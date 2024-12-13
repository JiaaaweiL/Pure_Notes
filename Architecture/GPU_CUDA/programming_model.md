### CUDA 的progamming model两点最重要：
- A way to organize threads on the GPU through a hierarchy structure
- A way to access memory on the GPU through a hierarchy structure
  
同时，可以从三个维度来看： Domain Level， Logic Level， Hardware Level  

Kernel: the code that runs on the GPU device. **CUDA 的编程模型是全异步的** When a kernel has been launched, control is returned immediately to the host, freeing the CPU to perform additional tasks complemented by data-parallel code running on the device. The CUDA programming model is primarily asynchronous **so that GPU computation performed on the GPU can be overlapped with host-device communication.**

### THREAD 组织模型：  
一个kernel所启动的所有线程被称之为grid，一个grid中的所有线程共享同一个global memory space，一个线程块是一组线程，可以通过 **线程块级别的共享内存，线程块级别的同步来互相配合，不同线程块之间的线程不能相互配合**    
线程通过参数来区分： BlockIdx(一个grid中的block编号） 和 ThreadIdx（一个block中的thread编号）  这些坐标是由CUDA运行时自动分配并初始化的内置变量  
```cpp
int nElem = 6;   
// define grid and block structure   
dim3 block (3);   
dim3 grid  ((nElem+block.x-1)/block.x); //floor division, (6+3-1)/3 = 2 block, each block contains 3thread, total 6 thread   
//最后一行这玩意相当于向上取整   
```   
对于一个dataset，一般的步骤是：先决定grid dimision 和block dimision，然后决定block size，然后再基于data size和block size决定grid dimision。      
对于block dimision，需要考虑kernel的性能，和GPU硬件的影响。      
**自己的理解： 因为一个kernel所启动的线程都放在一个grid中，所以我们不能选择有几个grid，只能选择grid中有几个block，以及block有几个thread**   




