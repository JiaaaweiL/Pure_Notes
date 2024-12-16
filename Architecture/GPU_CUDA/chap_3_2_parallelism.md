### Case Study： 一个二维的matrix add
```cpp
$ ./sumMatrix 32 32
 sumMatrixOnGPU2D <<< (512,512),  (32,32) >>> elapsed 60 ms
 $ ./sumMatrix 32 16
 sumMatrixOnGPU2D <<< (512,1024), (32,16) >>> elapsed 38 ms
 $ ./sumMatrix 16 32
 sumMatrixOnGPU2D <<< (1024,512), (16,32) >>> elapsed 51 ms
 $ ./sumMatrix 16 16
 sumMatrixOnGPU2D <<< (1024,1024),(16,16) >>> elapsed 46 ms
$ nvprof --metrics achieved_occupancy ./sumMatrix 32 32
 sumMatrixOnGPU2D <<<(512,512),  (32,32)>>> Achieved Occupancy    0.501071 
$ nvprof --metrics achieved_occupancy ./sumMatrix 32 16
 sumMatrixOnGPU2D <<<(512,1024), (32,16)>>> Achieved Occupancy    0.736900
 $ nvprof --metrics achieved_occupancy ./sumMatrix 16 32
 sumMatrixOnGPU2D <<<(1024,512), (16,32)>>> Achieved Occupancy    0.766037 
$ nvprof --metrics achieved_occupancy ./sumMatrix 16 16
 sumMatrixOnGPU2D <<<(1024,1024),(16,16)>>> Achieved Occupancy    0.810691
$ nvprof --metrics gld_throughput./sumMatrix 32 32
 sumMatrixOnGPU2D <<<(512,512),  (32,32)>>> Global Load Throughput  35.908GB/s
 $ nvprof --metrics gld_throughput./sumMatrix 32 16
 sumMatrixOnGPU2D <<<(512,1024), (32,16)>>> Global Load Throughput  56.478GB/s 
$ nvprof --metrics gld_throughput./sumMatrix 16 32
 sumMatrixOnGPU2D <<<(1024,512), (16,32)>>> Global Load Throughput  85.195GB/s
 $ nvprof --metrics gld_throughput./sumMatrix 16 16
 sumMatrixOnGPU2D <<<(1024,1024),(16,16)>>> Global Load Throughput  94.708GB/s
$ nvprof --metrics gld_efficiency ./sumMatrix 32 32
 sumMatrixOnGPU2D <<<(512,512),  (32,32)>>> Global Memory Load Efficiency 100.00% 
$ nvprof --metrics gld_efficiency ./sumMatrix 32 16
 sumMatrixOnGPU2D <<<(512,1024), (32,16)>>> Global Memory Load Efficiency 100.00%
 $ nvprof --metrics gld_efficiency ./sumMatrix 16 32
 sumMatrixOnGPU2D <<<(1024,512), (16,32)>>> Global Memory Load Efficiency 49.96% 
$ nvprof --metrics gld_efficiency ./sumMatrix 16 16
 sumMatrixOnGPU2D <<<(1024,1024),(16,16)>>> Global Memory Load Efficiency 49.80%
```
第一个配置，每个Block里有32个warp，第二个配置有16个warp，但是第二个配置有更高的Achieved Occupancy，是因为第二个配置的每个SM能够容纳更多的Block，所以拥有更多的available warp去调度来遮盖latency。
- Because the second case has more blocks than the first case, it exposed more active warps to the device. This is likely the reason why the second case has a higher achieved occupancy and better performance than the first case.
- The fourth case has the highest achieved occupancy, but it is not the fastest; therefore, a higher occupancy does not always equate to higher performance. There must be other factors that restrict performance.

第三组数据和第四组数据：load多不一定好，还要看load efficiency：request global load throughput/required global load throughput。

### Avoiding Branch Divergence






### Dynamic Parallelism
核函数可以在 GPU 内部调用另一个核函数，实现嵌套执行（Nested Execution）。父核函数（Parent）启动子核函数（Child），无需主机端（CPU）干预。    
父网格（Parent Grid）：由主机启动， 子网格（Child Grid）：由父网格的线程动态启动。  
执行规则：  
父网格必须等待所有子网格执行完毕后才能完成。  
线程块的执行也不算完成，直到所有线程启动的子网格都完成。  
如果线程没有显式同步，CUDA 运行时会隐式同步，保证所有子网格执行完毕  

共享全局内存和常量内存：  
父网格和子网格共享同一个 全局内存 和 常量内存。   
局部内存和共享内存是独立的：父子网格的局部内存和共享内存彼此隔离。  
内存一致性保证：在两个时间点保证内存视图一致：子网格启动时：父线程的全局内存操作对子网格可见，子网格完成时：子网格的全局内存操作对子线程可见，但需要显式同步   




