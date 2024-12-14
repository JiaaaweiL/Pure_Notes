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
对于一个dataset，一般的步骤是：决定grid dimision 和block dimision。先决定block size，然后再基于data size和block size决定grid dimision。      
对于block dimision，需要考虑kernel的性能，和GPU硬件的影响。      
**自己的理解： 因为一个kernel所启动的线程都放在一个grid中，所以我们不能选择有几个grid，只能选择grid中有几个block，以及block有几个thread**   

![image](https://github.com/user-attachments/assets/56d9033b-9148-4fa7-aeac-9f6f7298a13d)

### Debug
其实不好debug，因为永远是asynchronous调用，错都不知道错哪里，不过可以用如下的宏  
```cpp
#define CHECK(call)                                                       \
 {                                                                         \
   const cudaError_t error = call;                                        \
   if (error != cudaSuccess)                                              \
   {                                                                      \
      printf("Error: %s:%d, ", __FILE__, __LINE__);                       \
      printf("code:%d, reason: %s\n", error, cudaGetErrorString(error));  \
      exit(1);                                                            \
   }                                                                      \
 }
```  
像这样调用： CHECK(cudaMemcpy(d_C, gpuRef, nBytes, cudaMemcpyHostToDevice));

### Index Matrix with Blocks and Threads
第一种：2D的index： the matrix size is nx * ny, each thread doing one add.    
ix = threadIdx.x + blockIdx.x * blockDim.x  
iy = threadIdx.y + blockIdx.y * blockDim.y;  
idx = iy * nx + ix;  
![image](https://github.com/user-attachments/assets/986a2be0-bb95-4469-8404-2568cf216e17)   

例子：假设我们有8*6的矩阵，48个元素相加
我们先决定block size，每个block size 包含8个thread，即：4*2，于是乎，blockDim.x是4， blockDim.y = 2,
然后我们这个grid就需要x轴上2个block， y轴上3个，所以一共是6个block。
![image](https://github.com/user-attachments/assets/97df7a38-f745-4335-8464-e2b45700a0ce)

例子2：nx ny都是2<<14, 即16384*16384个元素  
有三种 lunch：  
sumMatrixOnGPU2D <<<(512,512), (32,32)>>> elapsed 0.060323 sec     
sumMatrixOnGPU2D <<<(512,1024), (32,16)>>> elapsed 0.038041 sec 比第一种好，因为直觉是两倍的block size，所以更多的并行        
sumMatrixOnGPU2D <<<  (1024,1024), (16,16)  >>> elapsed 0.045535 sec 所以更多的 并行不一定会带来性能的提升   

第二种，用以为的gird和block来处理二维的Matrix nx ny都是2<<14, 即16384*16384个元素    
```cpp
__global__ void sumMatrixOnGPU1D(float *MatA, float *MatB, float *MatC, 
      int nx, int ny) {
   unsigned int ix = threadIdx.x + blockIdx.x * blockDim.x;
   if (ix < nx ) {
      for (int iy=0; iy<ny; iy++) {
         int idx = iy*nx + ix;
         MatC[idx] = MatA[idx] + MatB[idx];
      }
   }
 }
sumMatrixOnGPU1D <<<(512,1), (32,1)>>> elapsed 0.061352 sec
``` 
上面的代码会generate 512个block，每一个block有31个thread，所以一共有即16384 个thread。   
可以看到，每一个thread负责的是 一列（row）的元素，也就是16384个元素。矩阵的存储应该是row domain   
换一种方法：我们有： 
sumMatrixOnGPU1D <<<(128,1),(128,1)>>> elapsed 0.044701 sec   
 ➤ Changing execution confi gurations affects performance.    
 ➤ A naive kernel implementation does not generally yield the best performance.   
 ➤ For a given kernel, trying different grid and block dimensions may yield better performance.   

