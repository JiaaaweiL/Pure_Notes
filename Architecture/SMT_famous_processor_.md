# 锐评SMT处理器
## Intel Pentium 4
![image](https://github.com/user-attachments/assets/f5f956e7-cfbc-4f02-bde5-f2326e0fc1fc)   

第一款商业化的通用SMT处理器（Intel的名字叫做Hyper-threading） 超标量，单核心，乱序，两个硬件上下文处理器。  
支持两种工作模式，单线程及多线程。   
资源划分是静态的，意味着，单线程情况下可以用所有的资源，而多线程情况下各用一半。坏处是浪费性能，好处是性能隔离好，单线程性能不受影响。    
Uop队列，指令队列，ROB都是静态划分的。physical architecture register是统一的。    
性能增加了15~27%在多线程和多任务场景下。 虽然SMT硬件开销小，但是资源共享和征用带来了设计复杂性。    

## IBM Power series
![image](https://github.com/user-attachments/assets/a1ecc8d4-1624-4730-8ef4-1b385cdd8f14)  
双核+SMT。每个核心支持两个物理上下文，每一个周期，一个核心最多从一个thread中取出最多8条指令放入两个hw thread buffer之一。 dispatch阶段将一个thread instruction buffer的最多五个指令打包，并且一起提交（像不像GPU?Warp）。这些指令在renaming 阶段并不会混合，只有在被发射到FU阶段才会被混合。   
设计目标之一是保持和Power4的兼容性。让软件优化仍然适用。并且Power5在增加SMT的情况下没有增加额外的pipeline register。
额外增加（比起power4）： PC， 额外的rename mapper， 完成逻辑的复制， 寄存器文件的增加（从 80 个整数寄存器增加到 120 个，从 70 个浮点寄存器增加到 120 个）  

Power5 提供了明确的线程优先级支持。尽管每个周期只能从一个线程中调度指令，但通过线程优先级，处理器可以决定每个线程获得调度周期的频率。提供了接口给Linux来执行优先级。线程优先级还能减少系统资源浪费（如果某个线程空循环/等待同步的时候下降优先级能够提高资源利用率）   
Power5 的硬件有能力控制线程之间的调度速率，以防止一个线程在停滞时占用过多的管道资源。例如，当一个线程由于缓存未命中或同步等待而停滞时，硬件可以通过以下几种机制减少其资源使用：     
- 降低线程优先级：如果某个线程占用了过多资源，硬件会自动降低该线程的优先级。    
- 停止取指：当线程遇到太多缓存未命中时，硬件可以停止从该线程取回指令。   
- 刷新指令：当线程由于同步等待（如等待锁）而导致无限延迟时，硬件可以刷新该线程的指令。

这些机制分别与 ICOUNT（基于资源使用的调度算法）以及 STALL 和 FLUSH 机制类似，用于减少线程资源争用带来的负面影响。    
Power5 也有单线程模式。Power5的资源是动态共享（而不是静态划分，像Pentium 4）但是物理寄存器不是。PA再单线程下不完全释放（说白了一定预留一组arch reg给另外一个线程。估计在写的时候binding with ThreadID）。   
Power6： 双核多线程，但是不是乱序执行，是顺序执行。（因为是顺序所以CR短Freq高）顺序执行+SMT来提高Throughput（怎么办到的？所以还是超标量？+不乱序做SMT???) 去读论文然后看架构吧？ 然后也是用指令打包成组发射执行。每一组最多五条。两个线程组最多七条，一起发射（超标量），（GPU既视感）。  
Power7： 继续OoO。 在每个核心上支持 4 个线程（比 Power6 的 2 个线程更多），并增加了超标量执行宽度（6 发射单元）。支持单线程，双线程，和四线程模式。   
在 ST 和 SMT2 模式下，每个线程的寄存器文件和流水线是共享的，允许线程灵活地将指令分派到任何流水线中。在 SMT4 模式下，寄存器文件和流水线是独立的，分别服务于两个线程。这种设计允许 Power7 在不同模式之间动态切换，从 **集群架构（Clustered Architecture多个线程共享流水线）切换到 结合核心架构（Conjoined Core Architecture多个流水线分别服务不同的线程）（INTERESTING!)** 看论文去吧靠北！  
[Power7 论文链接](https://www.researchgate.net/profile/Michael-Floyd-4/publication/220290623_Power7_IBM's_Next-Generation_Server_Processor/links/00463515dcf82abd08000000/Power7-IBMs-Next-Generation-Server-Processor.pdf)

# AMD推土机
后面来补充...
