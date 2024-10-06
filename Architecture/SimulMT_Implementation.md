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














