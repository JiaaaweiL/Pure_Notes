# Simultaneous MultiThreading
允许多个线程在同一个周期内向执行单元发射指令，从而最大限度地利用处理器的发射带宽。SMT 的设计旨在解决单个线程无法充分利用处理器资源的问题，尤其是当一个线程的**指令级并行性**不足以填满处理器的执行带宽时，可以通过执行其他线程的指令来补充这些空闲资源。    
难以识别“占有者”: 流水线的所有阶段和资源，包括寄存器文件、指令队列、重命名逻辑等，都是共享的，所有线程都可以在任意时刻访问这些资源。资源并不专属于某个特定线程。    

## 一个 SMT 的 Reference Implementation: MIPS R10000
![image](https://github.com/user-attachments/assets/cc6273ea-1a9c-425b-a134-3868577cd973)  
4宽，动态发射的超标量处理器, VIPT 缓存， 三个instruction queue（浮点，整数，load/store），前端是寄存器重命名+active list。  
重命名是register map（rat） + freelist（physical register not currently assigned to an architectural register）。 基本思路跟之前的乱序多发射一致。    
为什么SMT和OoO强绑定？ 1. 时间上的契合， 研究SMT的时候正好向OoO过渡  2. 设计的契合（边际效用小）   

取指阶段的改变： 如果从很多同时从很多thread的PC中取值，会极大的增加icache的设计复杂度。因此，取值都是取thread中的一个子集的指令（例如2）。如果多个线程取回指令，每个线程取的指令较少，单个线程内的依赖关系也会减少，寄存器重命名逻辑的复杂性就会降低（意思是，加入从一个thread取值，一个thread指令中的dependency会变少）。  
SMT 需要处理来自多个线程的指令，因此必须配备一个更大的物理寄存器文件。  理论上，物理寄存器的数量应该是支持的线程数量*ISA定义的寄存器数量 + ROB entry。 重命名器件里面，必须维持线程对应数量的RAT（例如有2个thread，就必须要有32*2的entry），RRAT同理。

### Fetch 阶段
分支预测其可以不该，但是SMT会降低分支预测器的中奖概率（怎么改？不同的thread不同的BP，或者将thread ID给binding 进去）。 TLB必须支持映射到不同的thread（废话，不同thread之间当然不能用一套TLB为了保证安全）。 Thread select logic和FGMT相似，我们可以fetch from one thread per cycle。但是，对于OoO机器，仔细的选择thread是提升性能的关键，因此很多时候有一些线程会占用更关键资源很长一段时间，有些分支指令的recovery会刷掉很多指令（所以要延后处理高风险的指令？为什么？早执行晚执行不都是执行？）    
别的微架构，例如Power5（IBM），线程选择发生在Fetch之后。IBM Power5会从每一个线程取指令，然后放进对应线程的buffer。 然后由指令选择逻辑去选择哪一个单元的指令进行解码。传统的结构是在取指令之前就已经决定了从哪个线程取指令，而IBM是会从每一个线程取指令，然后放入缓冲区。由指令选择器去选择从哪一个缓冲区取指令解码。这是他们的区别    
取指碎片化问题： 假设出现了一个branch，导致了即将要取的指令不在Icache里面，然后在接下来的周期里面导致了取指带宽下降很多（因为cache miss)    
解决思路：从两个线程中同时取指令（双端口的cache）    



### Decode/Renaming 阶段
寄存器重命名的一个重要功能是消除不同线程之间的寄存器冲突： 每个线程都有一组自己的寄存器，他们必须被映射到不同的physical reg。因为有了renaming， 在流水线的后续阶段（例如调度阶段和执行阶段），调度器和寄存器文件只需要关心物理寄存器名称，而不需要再参考线程 ID（TID）。   
RAT的数量是architectual reg * number of thread， 处理器会将 TID 和架构寄存器名称组合在一起，来确定该架构寄存器的物理映射。   
引入了SMT（增加了RAT entry之后，renaming 可能成为最新的critical path。 但是本质上RAT作为映射表增加不了很多面积。  
ROB也应该随之增加（要么独立，一个thread 一个ROB，要么统一ROB，与threadID Binding）  

### Issue/Reg File 阶段
Issue queue应该不怎么变，要增加branch recovery时，根据线程来flush指令的能力（无论做不做EBR），根据Instruction ID新旧来刷新。      
RAT的数量是architectual reg * number of thread。已经说过了，更多的thread肯定会要求增加physical reg的数量的。     
有一个问题是：增多physical reg的数量会导致access latency变慢。 Tullsen et al. [1996] show that an extra register stage costs single-thread execution 2% on average. 但是alpha21464并没有发现这个问题？ R10K的基准数据是半个CC能够读写一个reg。     

### 






