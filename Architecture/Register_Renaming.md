# 寄存器重命名：    
主要目的：解决指令的假相关性（WAW WAR)    
假相关性为什么存在？因为 1）有限个数的寄存器， 2） Loop猛猛的写入值   3）代码重用（造成跟2一样的问题）   
1. 早期的X86ISA中只有8个通用寄存器。导致了很多地方必须重复使用寄存器，造成很多WAW/WAR  
2. Loop会编译成猛猛往特定寄存器写值。产生大量WAW。虽然可以用unrolling来解决问题，但是unrolling factor很大的情况下总会用完。而且unroll会增加代码量（Icache-miss 增高）  
3. Core reuse，问题参考2

既然是由于有限个数的寄存器导致了false dependency，因此解法将会变得相当的明确：增加**指令集**中的寄存器的个数。 更改ISA会带来大量的兼容性问题，所有程序都需要重新编译。   
所以最好的方法是使用硬件管理寄存器重命名。定义更多的physical register，然后用RAT（RRT, Register Renaming table）做映射。 RAT可以用SRAM/CAM来实现。保持一个fre list标记那
些物理寄存器是空闲的，可以被拿来占用做重命名。   
### 三种重命名方式：
1. 将Architecture Reg File扩展来实现寄存器重命名  
2. 使用统一的Physical Reg File来实现寄存器重命名  
3. 使用ROB来实现处理器重命名。  

MIPS多采用方法2， Intel多采用方法3。 一般需要考虑以下的内容：
1. 什么时候占用physical register， physical register 来自哪里？  
2. 什么时候释放physical register， physical register去往哪里？
3. BP mis predict的时候如何处理？    
4. exception的时候如何处理？

### 1. 使用ROB进行寄存器重命名
将ROB作为了物理寄存器，在其中存储着所有speculative结果，而用Architecture Reg File存所有正确的结果。Super-scalar处理器中，每条指令会被按序放入ROB。指令结果被FU计算出来之后
也会被写进ROB。 但是因为有branch和exception的原因，这些结果未必正确（所以被成为speculative），在被commit之前，他们会一直呆在ROB中。直到commit，才会把ROB里面的寄存的计算完
的值写进ARF中。这种方式相当于将Physical register file和ROB的集成。   

另外一种是：当一个指令被写道ROB的Entry中，这个表项在ROB中的编号也就成了这条指令的目的地寄存器对应的物理寄存器。这样子就简历了将一个逻辑寄存器和ROB中的一个表项的编号建立了
映射关系。 所以只要ROB中有空闲的空间，寄存器重命名就可以一直进行。Speculative的阶段，只是将结果写进ROB。 指令被commit（retirement）的时候才会被写进ARF。    
RAT里面会有entry指示这条指令的结果是在ARF中还是在RAT中。而retire的时候，将会把数据从ROB中搬运到ARF中，造成了一个寄存器在**生命周期内会有两个存放位置**， 会对指令的**操作数读取**造成负面影响
（这就是问题所在！进来两个RS，还得知道去ARF还是某个ROB Entry中寻找数字）。而且会要求多一次的value搬运（把数据从ROB中搬运到ARF中）。因此，在实际的处理器当中，都会配合DATA-CAPTURE)的
issue 方式。   
除此之外还有两个缺点：
1. 很多指令没有目的地寄存器（br， store） 但是仍然占有了ROB，而且ROB都必须要有一个entry存value。 也就是说**即使指令没有RD，也没法省去ROB中的物理寄存器**  
2. 对于一条指令，既需要从ROB中读数， 也要从ARF中读数。导致ROB和ARF有一大堆端口，而又不是每时每刻会用到。

但是这种方法容易实现，设计复杂度也不高。Intel的P6架构的所有处理器都能见到他的影子。  

### 2. 将ARF扩展进行重命名
解决的问题是：ISA的有一些指令不需要重命名，例如branach，例如store。 如果采用上述的方法会浪费ROB的空间。因此可以将Physical Reg File独立成一个部件。 之前是将value存在ROB里面，现在就是将ROB里面的Value换成Physical register 的index，然后在做一次检索。这种方法还是需要一个renaming table。只不过之前记载的是ROB entry，现在记载的是Physical Reg File 的pointer。  

### 3. 使用统一的PRF进行Reg renaming（ERR，我写的）
将上述的ARF和PRF进行了合并，统一称之为PRF。 其中存储了所有的speculative/retire的寄存器值。 很显然尊崇Physical reg = rob + architectural的逻辑。    
两个RAT，一个负责解码后的AR到PR的映射，一个负责commit后AR到PR映射（叫RRAT, Retired RAT）。 外界的观察窗只能观察到RRAT映射的PR。    
核心问题是，一个reg合适能编程空闲状态呢？ 最简单的方法是，同一个arch RD被退休的时候，释放上次占用的pd。   
详细内容参照411。     
优势是： 寄存器的值只需要被写入一次，不需要再进行移动。再另外两种方法中，一个寄存器的值再生命周期之内有两个地方要存放。第一次写进ROB或者PRF中，第二次写道ARF中。所以基于ROB的是read/write hungary的。除此之外，前两种方法中，有两个地方（ROB/ARF）能存储源寄存器的值。所以需要更复杂的连线和通信。  

## RAT，重命名映射表
一个表格，使用Archi寄存器来寻址，得到对应的physical reg 编号。 物理实现上，要么用SRAM（sRAT），要么用CAM(cRAT)。

### 基于SRAM的重命名表：
对于每一个branch，都要做一个check point。对应关系不需要任何的标识位(用CAM可以增添标识位)。 当需要对分支指令的状态进行checkpoint保存时，需要将整个SRAM都保存起来。因此问题是：占用很大的面积。因此checkpoint数量不会很多。 例如，MIPS:R10K 一共有4个checkpoint。 为了知道那些physical register是空闲状态，还需要一个free list来记录。 新写道SRAT的值会覆盖掉原来旧的对应关系。可以保存到ROB中（这种方法是 用checkpoint然后恢复FREE LIST，不是411的free list恢复）（**当初411是怎么解决的？？有点忘了？**）     
现代处理器中大量的使用了预测算法。如果branch的预测是正确的，使用checkpoint就是浪费。那么，其实可以对低风险的指令不做checkpoint备份，只对高风险的branch做checkpoints。不做checkpoint（意味着不用EBR）然后flush到整条流水线，或者交由OS当成异常处理？     

### 基于CAM的重命名映射表
CAM不会对地址进行直接解码，而是将地址与储存器中每一个entry的内容进行比较，比较是相等的entry的地址是最终的结果（像fully associative cache）。在基于CAM的CRAT中，表象的个数等于物理寄存器的个数，使用Arch的编号对cRAT进行CAM寻址。每个ArchReg在cRAT中只能由一个有效的表象能与之对应（一个archi reg 能在cRAT中出现很多次，但是必须只有一个是valid)。每次进行checkpoint的时候，就保存一个有效位即可。   
![image](https://github.com/user-attachments/assets/33068e59-a034-49fd-b3ad-e5296399873a)   
每多一个checkpoint，只需要多出一个bit column。 所以能允许很多很多个checkpoint。 对于写cRAT，只需要将其作为一个普通的储存器，使用物理寄存器的编号作为地址就可以了。  

一条分支指令在进行重命名之前，需要将cRAT中的V bit复制到CP中，完成checkpoint。 当后续阶段完成后， （如果发现mis predict）直接将CP中的写回valid bit就行了。 




