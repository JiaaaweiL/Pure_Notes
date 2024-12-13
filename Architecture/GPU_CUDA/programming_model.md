### CUDA 的progamming model两点最重要：
- A way to organize threads on the GPU through a hierarchy structure
- A way to access memory on the GPU through a hierarchy structure
  
同时，可以从三个维度来看： Domain Level， Logic Level， Hardware Level  

Kernel: the code that runs on the GPU device. When a kernel has been launched, control is returned immediately to the host, freeing the CPU to perform additional tasks complemented by data parallel code running on the device. The CUDA programming model is primarily asynchronous **so that GPU computation performed on the GPU can be overlapped with host-device communication.**
