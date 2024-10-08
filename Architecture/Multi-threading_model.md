# 多线程体系结构: 
定义：一个核心可以在没有软件帮助的情况下执行多个指令流。 传统的CPU如果想要切换线程必须：  
1. 软件陷入（应该是OS CALL)    
2. 然后花费数千个CC保存现态，将其存入内存（看OS）   
3. 将另外一个线程拉近CPU    
多线程CPU可以将线程的状态保存进入Core/临近Core，可以达到快速转换的目的，潜在的发挥出CPU的能效。  
为了达到这个目的，多线程CPU必须能够保存多个线程的状态。我们将其称之为“hardware contexts” hardware context的数量代表支持的multithreading 的等级。（这里表示的是 多少条线程可以在软件不接u人的情况下切换。 线程状态通常由，PC, CPU寄存器的值，特殊目的的寄存器的值。（参考OS）

硬件MT的好处：
1. **硬件多线程有助于解决硬件支持的指令级并行性ILP与线程实际可以执行的 ILP 之间的差异**。通俗来说，如果处理器硬件有执行多个指令的能力，但单个线程无法提供足够的指令来让处理器满负荷运行，硬件多线程可以让多个线程共享处理器资源，从而提高利用率。这种多线程架构也适用于标量机器（非并行处理器），可以弥补硬件带宽和单个线程实际吞吐量之间的差距。   
2. **线程级并行性TLP**：多线程可以通过线程级并行性（TLP）来提高处理器利用率。当一个线程无法充分利用处理器资源时，其他线程可以利用这些闲置资源。      
3. **硬件多线程实现的虚拟化**：硬件多线程使处理器虚拟化，表现得像一个多处理器系统。通过增加第二个硬件上下文，运行在线程中的任务会感觉像是运行在一个虚拟核心上，这个核心具有原始核心的硬件能力，减去第一个线程所占用的资源。通过这种方式，我们可以利用未使用的资源构造虚拟核心，从而在一定程度上达到多核处理器的性能，但其面积和实现成本却要低很多。

Multithreading的原始目的： 补齐硬件并行和软件并行之间的差异。 在早期计算机中，硬件和软件并行性之间的差距相对较小，除了在进行I/O或长时间存储（如磁盘）的操作时，会出现处理器空闲的情况。在引入时间共享技术后，软件多线程（Context switch）能够有效隐藏这些延迟。之后，这些差距就被内存速度和cache补齐。 计算的多样性继续引起了这些问题（有的复杂操作要花费大量的时间，有的操作只需要几个CC）    


## Execution model
multi-thread processor 的定义是：一个能够处理多条指令流，而不需要软件介入的处理器。  这意味着处理器内部可以在硬件中保存多个PC，从而能够跟踪多个指令流。   
- SMT: 一个core同时处理多个thread
- CMP: chip multiprocessing 是在一个芯片上集成多个独立的处理器核心，每个核心可以独立运行不同的线程。(奇奇怪怪）
多线程技术的目的是解决处理器在单线程被阻塞或暂停时无法充分利用执行资源的问题。特别是在超标量处理器（Superscalar Processor）中，当一个线程的执行暂停或效率低下时，处理器资源会闲置，从而浪费了执行潜力。

- 多线程的目的：**解决浪费**  解决单线程被阻塞或暂停时无法充分利用FU的浪费。特别是在Superscalar Processor中，当一个线程的执行stall或FU要multi cycle然后RAW反压上游issue时，处理器资源会闲置，从而浪费了执行潜力。
- 将资源浪费分成 vertical 和 horizontal：

  ![image](https://github.com/user-attachments/assets/7ee6d245-4c1f-4f10-b6c1-51ff6b461047)     
一行四个是四个FU。 空白表示正在浪费 Vertical指处理器在每个周期中，无法充分利用其所有的指令执行单元， horizontal指的是在流水线的不同阶段中，由于指令之间的依赖性或者等待内存等原因。      
仔细想一想，除了引入SMT之外还能用什么来解决？ 更深的ROB->更多的ILP能够解决。但也有marginal effect， 只能解决一些些。     
- horizontal 浪费： 当指令之间存在依赖性，导致处理器无法充分利用其发射带宽时，就会产生水平浪费。   
- Vertical 浪费： 垂直浪费发生在整个流水线由于某种原因（如长延迟事件、数据依赖等）暂停工作，导致处理器在某个周期中没有发射任何指令。（大概率是store或者长时间的ALU指令干的）

MTprocessor要求保存多个上下文的状态。最重要的上下文状态是寄存器。有的MTmodel要求所有的寄存器都随时有效，而有的要求存储单元可以off core。 这些要求可以是一个MT的CPU的核心取舍。Architecture 寄存器的位置上最核心的成本性能取舍。    


**Chip multiprocessor**，又叫做multicore Processor， 是一种多线程的延续。 在Chip multi-processor中，核心资源，甚至L1都不共享。只有L2, L3, 互联网络，甚至是内存控制器是共享的。CMP 处理器的设计允许多个核心在同一个周期内同时执行不同线程的指令，因为每个核心都分配有自己的执行资源。  
chip multiprocessor没有能力去改变垂直浪费和水平浪费。但是这种浪费都会减少，因为发射带宽是平均静态分配的。因此每个核心的浪费对整体处理器的影响较小。   

比如，在一个4核的 CMP 中，如果每个核心是**2发射超标量** 架构，总的发射带宽是8。当一个线程被阻塞时，最多只有两个指令周期（来自一个核心）的发射带宽被浪费。  这种静态划分虽然限制了资源利用的灵活性，但带来了更好的隔离性和性能的可预测性，因为每个核心的资源是完全独立的。 

在 CMP 中，寄存器状态是分布在各个核心中的。因此，每个核心的寄存器文件的设计不比单核处理器的寄存器文件更复杂。    

**Conjoined Core Architectures**  
联合核心（Conjoined Cores） 是指多个核心**共享部分但不是全部执行资源的架构**。与传统的 CMP（芯片多处理器）不同，CMP 每个核心的资源是完全独立的，而在联合核心架构中，核心之间可以共享某些特定的资源，从而提高这些资源的利用率。  
这种架构设计的目标是模糊传统的核心边界，以便根据需求共享不常被单个核心充分使用的资源，从而最大化利用率和效率。  
可共享的资源包括L1/L2缓存， 浮点单元等等。 Conjoined 资源共享的具体方式上决定了其与 CMP 在应对水平浪费和垂直浪费上的相似性。如果浮点单元是共享的，而整数流水线不是共享的，那么浮点单元可以用来减少垂直浪费(为什么是垂直的？因为共享了资源，不用推迟CC？)，但整数单元则无法应对核心内的浪费。  
例子：臭名昭著的AMD 推土机。  
[一点黑话：为什么推土机这么TM菜！](https://www.reddit.com/r/Amd/comments/5q91tn/what_made_the_bulldozer_architecture_so_bad/)    

### Coarse-Grain multithreading
粗粒度多线程， AKA block multithreading or switch-on-event multithreading。 一个计算核与很多个hardware context相绑定。hardware context可以是PC， regfile等等。 但是只有一个硬件context可以获取上下文， 所以，我们仅仅可以使用context switch来掩盖某个指令流的latency。 本质上来说，Coarse grain的multi-thread的模式很像很像软件多线程，只是硬件多线程context switch的速度会快很多很多。 软件多线程能够隐藏的非常长的延迟，比如说进disk。然而硬件多线程可以handle例如cache miss， 等待barrier之类的延迟和。    
Coarse grain的多线程不能够作用于horizontal waste/长的vertical waste。 非执行的寄存器可以放置在核心更远的地方。只有正在执行的线程的寄存器会被放进寄存器内。   

### Fine-Grain Multithreading
细粒度多线程， interleaved multithreading， 也是讲多个硬件上下文关联到每个核心，但是上下文切换是没有延迟的。 因此， 它可以每周期的切换处理来自不同线程的指令（COARSE Grain需要浪费一些vertical cycle来做切换）。  
但是再一个流水线阶段里，只有一个线程的指令能够存在。    
FGM仍然不能解决水平浪费。但是能够消除垂直浪费（如果遇到一个等待时间超级长的，就直接执行另一个线程的指令）在超标量处理器带来之前，FGM是最激进的多线程模型。   
即便FGM机器只需要读取一个线程的寄存器，但是它可能会在下一个CC读取新的context的寄存器。 因此所有的寄存器必须距离流水线尽可能的近，虽然不存在同时读的情况。

### SMT, simultaneous multithreading
SMT 与粗粒度和细粒度多线程一样，拥有多个硬件上下文（包括程序计数器、寄存器文件等），使多个线程能够在同一个处理器核心上并发执行。     
无上下文切换：在 SMT 中，没有传统的“上下文切换”概念。与粗粒度多线程需要几个周期、细粒度多线程在每个周期切换线程不同，SMT 允许多个线程的指令同时存在于流水线的不同阶段，无需切换线程上下文。所有线程的指令都可以随时被发射和执行。   
解决垂直浪费：SMT 通过允许多个线程同时运行来隐藏长延迟（例如缓存未命中），当一个线程因延迟而无法继续执行时，其他线程的指令可以立即利用可用的执行资源。    
解决水平浪费：与细粒度多线程不同，SMT 可以在每个周期检查所有线程的指令，并寻找可以立即发射的指令。如果一个线程暂时没有可发射的指令，其他线程可以填补这个空闲的发射带宽。这意味着，任何线程无法使用的资源都可以自动被其他线程利用，从而减少水平浪费。   
由于 SMT 允许多个线程的指令同时发射并执行，因此所有硬件上下文中的寄存器文件必须能够快速访问并供流水线使用。这是为了确保在每个周期中，无论哪个线程的指令被发射，处理器都能够及时获取所需的寄存器数据。    


### hyper model: 
CMP: 当线程数等于核心数的时候，是最高效的。当线程数小于核心数是，资源没有被充分利用。当线程数大于核心数时，无法从MT中获得收益。因此CMP+SMT是一个好的组合，能够从线程数大于核心数中获得收益。 
CGMT+SMT: 延迟的双峰分布：处理器遇到的延迟通常呈现双峰分布：  
短延迟：由算术指令依赖性导致，例如寄存器依赖性和计算操作中的短暂等待。  
长延迟：主要由于缓存未命中等问题导致。  
SMT 对短延迟的优势：SMT 在隐藏短延迟方面非常有效，因为它能够在短暂依赖等待时，切换到其他线程执行指令。  
SMT 可能的过度开销：对于长延迟，如缓存未命中，SMT 可能效率过高，因为切换到其他线程的频率较高并不会显著提高性能。在这类场景中，粗粒度多线程（CGMT）则可能更适合。   
![image](https://github.com/user-attachments/assets/e8754713-633f-4532-80d2-ce1bc706ef18)      

### GPU和Warp scheduling
运用了SIMD的思想，和CMP是不一样的。 一个典型的config是将32个线程打包成一个warp作用于一小对核心上（8个）。 warp scheduling用时分复用的方法进行。32个thread， 8个核心，通常需要3个CC去执行完32个thread（这里是FGM, 假设thread 与 thread之间切换是立即的)












