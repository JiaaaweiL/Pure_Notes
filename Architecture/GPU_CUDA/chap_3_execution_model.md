## GPU overview
**GPU is built around a scalable array of Streaming Multiprocessors (SM) GPU  hardware parallelism is achieved through the replication of this architectural building block.**  
SM的核心组成：CUDA core,  Shared Memory/L1 Cache, Register File, Load/Store Units, Special Function Units, Warp Schedule  
- 一个SM可以支持数百个thread的同时执行，通常，GPU里有很多SM --> 数千个thread可以同时执行。
- Kernel grid启动的时候，线程块将会分配到available SMs上，分配后，threadblock里面的thread 只能在分配的SM上执行
- 一个SM在资源可用的情况下可以承载很多个thread block，一个thread block只能被分配在一个SM内
- 单个thread之内的指令可以pipeline，利用了ILP

执行规则： 
- 可执行的thread被以32的粒度打包成wrap，一个warp的thread将会同时执行相同指令
- 每一个thread拥有自己的pc，reg，data
- SM负责讲thread block拆成warp，然后在硬件资源可用的时候执行

**SIMD 和 SIMT的区别**： SIMD要求所有向量元素以统一的同步组执行，SIMT允许线程在warp中独立的执行。 SIMT要求warp中所有的thread有相同的起始地址，但是各个thread可以有不同的行为。   
The SIMT model includes three key features that SIMD does not:  
 ➤ Each thread has its own instruction address counter.  
 ➤ Each thread has its own register state.  
 ➤ Each thread can have an independent execution path.  
![image](https://github.com/user-attachments/assets/6655bb02-3514-450d-8237-ce7a74f76c0e)    
![image](https://github.com/user-attachments/assets/fd98f1a0-182a-42ba-b507-b18e6bdc6eda)  

shared memory 和register是SM的资源，将会被partition给每一个thread block。thread block里面的thread可以使用这些资源进行通信。 thread block里面的thread逻辑上是并行的，实际上并不是。因此，CUDA提供了block内的同步方式，没有提供block之间的同步方式。    

Thread block中的Warp可以被schedule in any order. 激活的warp取决于SM的资源限制。当一个warp是IDLE的时候，SM可以调用另外一个warp来执行。
- Warp之间的切换不会有很多overhead（开销）因为CUDA 的硬件设计中，每个 SM 的关键资源（如寄存器文件、线程上下文状态等）是 **预分配给所有正在运行的 warp 的**。 因为这些资源已经为所有 warp 分配和存储好，**切换warp 时不需要像 CPU 一样保存或恢复上下文（context switch）**，因此切换几乎没有开销。 
    
  
### Fermi 架构以及执行模型
Fermi一共有512个CUDA core，每一个cuda core有pipelined的int ALU 和FPU（floating point unit），可以每周期执行一个整数或者浮点数。 512个CUDA core被组合成16个SM，每个SM有32个CUDAcore. 用PCIE和CPU相连，然后用GigaThread Engine负责Thread block到SM的分配。thread block里面的thread到warp的组合是线性的：theadid 0到31是warp0， 32到63是warp1，以此类推。一个SM有两个warp scheduler和两个instruction dispatch unit。当一个thread block被指定到SM之后，两个warp scheduler选择两个warp，然后各发射一个指令去SM的一半的CUDA core。Fermi的一个SM可以同时处理48个warp，总共1536个thread（48*32=1536）。    
**因为每一个threadblock中的指令都是开始于同一个PC地址，所以所谓的，选择一条指令执行，其实是执行了整个warp，每一个thread的一条指令，也就是共计32条指令。** 其实是以warp为单位的交织执行，所以也就印证了上面说的：warp切换几乎没有开销，因为要藏延迟。      

![image](https://github.com/user-attachments/assets/5b961ebe-8a9d-46dd-8c7b-6106d02ac1d8)   

### warp divergence
当一个warp内的指令有if-else分支，从而，一个warp内的指令在两条path上，就叫做有warp divergence。   
推测是硬件行为， Warp 内线程的执行路径取决于运行时输入数据， Execution Mask： 硬件为 warp 内的每个线程维护一个执行掩码，用于标记哪些线程当前需要执行。  
当发生 warp divergence 时，硬件会分组执行不同的分支，每次激活一个子组的线程，同时禁用其他线程。  
```cpp
 __global__ void mathKernel2(void) {
   int tid = blockIdx.x * blockDim.x + threadIdx.x;
   float a, b;
   a = b = 0.0f;
   if ((tid / warpSize) % 2 == 0) {
      a = 100.0f;
   } else {
      b = 200.0f;
   }
   c[tid] = a + b;
 }
```
这里就没有 warp divergence，因为线程是以warp为粒度分叉的  如果改成，比如if (tid % 2 == 0) 就有问题了  Cuda编译器也有优化，例如替换成 x = y? a:b; 的形式  

### resource partitioning
一个SM里面能够同时容纳的thread block 取决于kernel需要的寄存器数量和shared memory数量。如果每一个thread需要很多的寄存器，SM能容纳的warp更少。如果可以减少kernel消耗的寄存器数量，则可以让更多的warp留下。 
**In order to maximize GPU utilization, you need to maximize the number of active warps**    
Number of Required Warps = Latency * throughput; 假设需要6个warp through put per cycle, 每一个warp塞进去要过5cc才会available，就需要30 = 5*6 个warp去填满latency；   
![image](https://github.com/user-attachments/assets/25b95e5c-cff7-460f-8093-b1da51c42d36)   
上面的的表格是对于full arithmetic operation而言的。   
下面是对于memory utilization的   
![image](https://github.com/user-attachments/assets/1838b002-0708-4881-a715-5a967114649d)    
92是 144GB/sec 除以 1.566GHz =92Bytes/sec。 memory latency 是800个CC，也就是说，至少保持800 * 92 Byte/CC = 74KB的请求在排队，才能吃满所有带宽（最大化带宽利用）。   
假设每个thread正在执行4Byte的daa movement，74KB/4BperThread = 18,500 Thread, 18500/32Thread per warp = 579 warp 来吃满带宽。    
Fermi有16个SM，也就是需要579warp / 16 = 36 warp per SM来吃满带宽。 32*32 - 1024， 也就是说如果每个thread只吃4byte，永远吃不满。   

### Occupancy
Occupancy 越高，意味着越多 warp 正在被硬件并发处理，越有可能达到 GPU 的理论性能。 occupancy = active warp/ maximum warps   
 ➤ Small thread blocks: Too few threads per block leads to hardware limits on the number of warps per SM to be reached before all resources are fully utilized.   
 ➤ Large thread blocks: Too many threads per block leads to fewer per-SM hardware resources available to each thread.   
假设有一大堆thread，当然可以有很多block，很多warp吃满所有SM上的资源。假设有限，则分情况讨论：如果是计算密集型，则集中于吃满少量的SM。如果是存储密集，则吃满所有的SM，充分的利用SM内部的资源。 （待讨论）  

### Synchoronization
- System Level： host会在launch kernel之后返回控制权，device和host之间是完全异步的，所以会有cudaDeviceSynchronize() lai wait until all cuda operation have completed.
- Block Level： block local barrier for all the thead within the block

There is no block to block synchoronization!




