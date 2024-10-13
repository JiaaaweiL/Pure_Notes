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

