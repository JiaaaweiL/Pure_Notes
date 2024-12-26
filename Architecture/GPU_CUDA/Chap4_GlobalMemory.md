# CUDA Memory Model: 
1. Programmable or Non-Programmable: 
2. Hierarchy: Reg -> Shared Memory -> Local Memory -> Constant Memory -> Texture Memory -> Global Memory

**Register**  
The automatic variable declared in a kernel without any other type qualifiers is generally stored in a register; On Fermi GPUs, there is a hardware limit of 63 registers per thread. If a kernel use more register than they need, will spill over to local memory.   

**Local Memory** Values spilled to local memory reside in the same physical location as global memory. Local memory has high latency and low bandwidth.   
- Local arrays referenced with indices whose values cannot be determined at compile-time.
- Large local structures or arrays that would consume too much register space.
- Any variable that does not fi t within the kernel register limit.

**Shared Memory**   
Each SM has a limited amount of shared memory that is partitioned among thread blocks.Therefore, you must be careful to not over-utilize shared memory or you will inadvertently limit the number of active warps.  每个流式多处理器（SM，Streaming Multiprocessor）上的共享内存是有限的，并且需要分配给多个线程块（thread blocks）过多使用共享内存会导致**活动warp（active warps）**的数量减少，从而降低程序的整体并行度和性能。  
The L1 cache and shared memory for an SM use the same 64 KB of on-chip memory, which is statically partitioned but can be dynamically confi gured at runtime using   

**Constant Memory**    
Constant memory resides in device memory and is cached in a dedicated, per-SM constant cache.   
当所有warp都从同一个地址里面读取信息的时候，constant memory的性能最佳  

**Global Memory**    
最大，延迟最高， 最常用。 不同线程可能会同时访问或修改全局内存中的同一地址，可能导致未定义的行为。特别是跨线程块（thread blocks）的同步是不可行的， 使用原子操作（atomic operations）避免数据竞争。
在需要跨线程块同步时，利用全局内存实现“手动同步机制”。  每次全局内存访问以事务的形式发生，大小为32字节、64字节或128字节。事务的起始地址必须是32、64或128字节的倍数，否则会导致性能下降。如果一个warp中32个线程的访问地址是连续的（如数组访问），可以极大减少事务数量。 反之，地址分布分散会导致更多事务，增加不必要的数据传输。  

**Cache**     
GPU caches are non-programmable memory. 64KB片上存储可以被config成shared memory或者L1。 Cache只缓存读，不缓存写。

**Pinned Memory**   
![image](https://github.com/user-attachments/assets/6722b269-897e-409d-8d25-3ef868ce5692)  
CPU的内存是pageable的。在Host -> Device 之前必须要做Pinned 保证内存不会在传输的过程中被换出， 提高传输的可靠性和性能。   
必须先分配Pinned Memory，然后做pageable memory -> pinned Memory 然后 Copy to host. 当数据 > 10 MB的时候会有优势，因为Pinned Memory的melloc会更慢。 

**Zero Copy Memory**    
Zero-Copy Memory 是一种特殊的主机内存（host memory），它：是 Pinned Memory（锁页内存），不会被分页到虚拟内存。   
被直接映射到 GPU 的设备地址空间，因此 GPU 可以通过设备指针直接访问主机内存，而无需显式的内存拷贝操作。  
主机内存和设备地址空间共享相同的物理内存区域。 允许 GPU 直接读取或写入主机的 Zero-Copy Memory，而无需通过 PCIe 拷贝到设备内存。 **但需要Device/Host 两者同步来避免未定义行为**
cons: 高延迟： 每次访问 Zero-Copy Memory，都需要通过 PCIe 总线传输数据，延迟远高于设备内存。    
低带宽： Zero-Copy Memory 的带宽受限于 PCIe 通道，通常比设备内存慢一个数量级。   
频繁读写性能低下： 对于频繁的读写操作，Zero-Copy Memory 的性能可能比显式拷贝更差。   
使用场景：   
数据量小，传输频率低：  如果数据量较小且无需频繁传输，Zero-Copy Memory 可以简化内存管理。   
实时数据传输：  主机和设备需要实时共享数据（如传感器输入），可以利用 Zero-Copy Memory。     
  
Zero-Copy Memory 更适合 小规模数据 的场景，因为它能省略 HtoD 拷贝步骤。   
普通设备内存更适合 频繁访问或大数据量 的场景，因为设备内存的访问延迟和带宽明显优于 Zero-Copy Memory。     

**Unified Virtual Addressing**
什么是 UVA (Unified Virtual Addressing)?  
通过统一虚拟地址空间（Unified Virtual Addressing），主机和设备内存共享一个统一的地址空间。  
在 UVA 模式下，主机和设备使用相同的指针访问相同的内存区域，无需再区分 "主机指针" 和 "设备指针"。  
pro:  
无须手动获取设备指针： 传统上，如果使用 Zero-Copy Memory，你需要通过 cudaHostGetDevicePointer 将主机内存的指针映射为设备可用的指针。   
使用 UVA 后，这一步骤被完全省略，主机指针和设备指针是相同的。   
透明的内存访问： 程序员不需要关心指针是否指向主机或设备，内存访问对应用程序代码变得完全透明。  
