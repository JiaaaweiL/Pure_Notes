# RTL LOGIC DESIGN
## Pipeline 相关内容，有那些hazard，pipeline有什么好处，坏处，为什么不用超级多层pipeline？
Hazard 相关：Data Hazard，Control Hazard, Structural Hazard     

- Data Hazard(AKA Data dependency). There are three kinds of Data dependency: Read after write, Write after write, Write after read. RAW is TRUE Dependency, WAR and WAW are false dependencies (can be solved by register renaming).   
- Control Hazard(AKA Branch Hazard). When there is a branch miss predict, we need to flush the pipeline and execute the correct instruction.  
- Structural Hazard: Many instructions trying to access the same hardware resource at the same clock cycle.  

Pipeline的好处：  
- boost the throughput, and increase the utility of the hardware resource. 
- The critical path will be shorter. Thus, the pipeline can boost the frequency (and performance).

Pipeline的坏处： 
- A deeper pipeline will increase the penalty when there is a branch miss-predict.
- A deeper pipeline will increase the power and area (more stage means more Dff), and the design complexity increase.

## Branch predictor/BTB 设计
- 最简单的 bimodal predictor

用pc的部分值去index一个saturating counter 最简单的做法，但是会被aliasing 困扰（两个不同的pc index 到同一个saturating counter，但是branch的方向不同，造成永远不饱和，叫destructive aliasing） 
变体之一是用哈希代替直接从PC中间取值 可以减少这种情况。  

- 基于局部历史的分支预测 local branch history   

上述的方法无法处理TNTNTNT,改进方法是是用一个branch history register hold每一个pc历史的分支数据，然后用这个历史状态来index saturating counter     
改进1是不可能为每一个分支指令配2一个BHR和PHT，因此是用PC的一部分来寻址PHT，相当于一部分PC要共用一个BRH。可以是pc哈希+pc部分位来index， 也可也是BHR+PC的位拼接    

- 基于全局历史的分支预测global branch history

解决的问题是： 一些分支指令的结果依赖于前面的分支指令，然后就用GHR global history register index PHTs 的entry。具体选择哪一个PHT，用PC做哈希决定。     
如果这种方法有很多的saturating counter, warm-up time(training time)过长，存储的bit过多，一个简单的方法是用一个PHT，然后用GHR+PC hash的位拼接。   
其实跟Local branch history很像。 

- BTB: 一个cache，一般是direct mapping或者少way的set associative BTB。将发生跳转的分支指令对应的目标指令放在BTB中。对pc-relative的指令，BTB的目标地址预测是准确的，但是对于间接分支，跳转不准确。  

 
## Memory + Cache hierarchy
为什么要有cache？ 无奈的妥协：储存器技术的发展速度没有处理器的发展速度快。  
![image](https://github.com/user-attachments/assets/88526f7d-3cd0-4085-9822-0f5340018c1e)   
Direct mapping, set-associative, fully associative 三种组成方式：地址的组成是：tag[] + set[] + offset[]   
offset bit = #bit to index the size of cache line. set bit = #bit to index the number of set. tag bit = 32bit - offset bit - set bit.    
因此， fully associative是没有set bit的，因为只有一个set. direct mapping 有最多的set bit.     
fully associative 有高的hit rate， direct mapping 有最低的latency，set-associative是中间的trade off. Direct mapping 的hit rate低，因为造成ping-ponging effect。 pattern是ABABABABAB，然后一直flush，造成0 hit rate。   

- Replacement Policy: Random, FIFO, Least recently used, Least frequently used, Least costly to re-fetch, Hybrid Replacement Policies

Modern processors do not implement TRUE LRU in highly associative cache. Because of the cost, and LRU is an approximation to predict locality.(理解！！！）     
True LRU会有set trashing 问题： 4way，然后五个轮流访问，hit rate 是0%； 因此我们需要Belady's OPT optimal placement policy. 选择未来最长时间内不会被访问的快，确保了最小的miss rate。但是不能确保minimun execution time，因为cache miss 的latency 和cost不同（有的从dram，有的从下级cache取。另外也没有考虑到MLP的情况，能overlap能够显著的减少执行时间。  

- 一个问题是：When do we write the modified data in a cache to the next level? Write-back/Write-through cache

Write-back cache:   
Can combine multiple writes to the same block before eviction,  Potentially saves bandwidth   

Write-through cache:      
Simpler design, All levels are up to date & consistent,   
More Bandwidth intensive no-combining of write  

- 一个问题是：Do we allocate a cache block on a write miss?

Allocate on write miss(一般是write-back cache): Can combine writes instead of writing each individually to next level; Simpler because write misses can be treated the same way as read misses      
Requires transfer of the whole cache block    

No- Allocate on write miss(一般是write through): Conserves cache space if locality of written blocks is low 

- 一个问题是false sharing： What if the processor writes to an entire block over a small amount of time?

subblocked cache： devide a block into subblocks; have separate valid bit and dirty bit for each subblock.   
No need to transfer the entire cache block into the cache,  More freedom in transferring subblocks into the cache 
Complexity in design, and May not exploit spatial locality fully 

- I-Cache  D-Cache separate or unified?

Unified cache的好处： Dynamic sharing of cache space -> better overall cache utilization   
坏处是：Instraction and data can trash each other(no guaranteed space for either)   
Instruction and data are accessed in diffierent place. Placement will be a problem(where to place for both fast access?)    


### Cache 的总体设计和考量：

**First-Level Cache**   
Small， lower associativity， latency is critical   
tag store and data store are usually accessed in parallel  
**Second-level Cache**    
Decisions need to balance hit rate and access latency    
Usually large and highly associative; latency not as important    
**Further-level (larger) caches**    
Access energy is a larger problem due to cache sizes  
Tag store and data store are usually accessed serially  
当从内存中加载数据的时候，放在那一集的内存？当从里层缓存换出的时候，换出到哪一层缓存？允不允许Bypassing？ Bypassing又要照顾到一致性问题？
Inclusive：里层的缓存的内容必然包含在外层缓存中：简化了一致性模型。  
Exclusive：里层的缓存不存在于外层缓存中：优化了缓存利用率


## Virtual Memory
The benefit of Virtual Memory: Protect, sharing, isolation and also simplifies the programming model;  
Virtual memory gives the illusion that the program owns the whole memory, and that the size of the memory is infinite. 
Physical Memory存在的问题：   
1. Limited size， 如果一个program needs more, what gonna happened? Should the programmer manage data movement form disk to the physical memory?   
2. Multiple programs may need the physical memory. Should the programmer make sure all processes can fit in physical memory?
3. Should the programmer ensure two processes do not unintentionally or incorrectly use the smae physical memory protion? 
4. Should the programmer ensure two process do not incorrectly use the same physical memory address?

**Difficulty of Physical Addressing:**  
1. Programmer need to manage physical memory space
2. Difficulty to support code and data relocation
3. Difficulty in supporting multiple processes
4. Difficulty in supporting data sharing across processes.

**Virtual Memory**: Give each program the illusion of a large address space. So that the programmer does not need to worry about managing the physical memory, basic mechanisms including indirection and mapping, using address translation mechanism maps this address to a physical address.     

**4 ISSUE**    
1. When to map a virtual address to a physical address? Ans: When virtual address is first referenced by the program.      
2. What is the granularity? Byte, KByte, GByte? Multiple granularity? Ans: 4KB(based on historical disk design)     
3. Where and how to store the VA -> PA mapping? Ans: Memory, page table, SW/HW OR cooperative.     
4. What to do when physical address space is full? Ans: Evict an unlikely-to-be-needed VM from physical memory.    

Method: Page Table: Mapping VA to PA.   
![image](https://github.com/user-attachments/assets/f1bd0015-abba-417e-a39f-8275191f3eb4)   
![image](https://github.com/user-attachments/assets/74eed96a-d413-4b89-872d-416fd6a7f624)

Page Table is located at physical memory address specified by the PTBR(Page Table Base Register).(CR3 in X86) We can access the PTE at address PTBR + VPN * PTE-size.      
### Page Table Issue
**Issue1 :** Page table is large, it will consume the memory space. **Solution:** Multi-level page table:  Organize page table in a hierarchical manner such that only a small first-level page table has to be in physical memory.    
Issue Example:  4 byte/entry is just the assume.       
![image](https://github.com/user-attachments/assets/f9a0df09-0212-4a10-8e00-33c763230df4)     
Solution Example: First Level page table must be in the physical memory. Only the needed second level page table can be kept in the physical memory.    
![image](https://github.com/user-attachments/assets/ad0c9b6d-8969-472f-946b-cad811d5e3af)        
**Issue2:** Each instruction fetch or load/store requires at least two memory accesses, The number of memory accesses increases with a multi-level page table. **Solution** Cache the Page Table Entries (PTEs) in a hardware structure in the processor to speed up address translation. The cache of PTE is called TLB. Translation look-aside buffer        
Small Cache of most recently used VA to PA translation. Small: few cycle latency, typically 16 - 512 entries at level 1(这tm个TLB还做multi level cache？没必要吧？) Usually high associativity,  >= 90-99% hit rate typically. Reduce the memory access for most instruction fetches and LD/SD to only one TLB access.      
![image](https://github.com/user-attachments/assets/898f8b0a-507f-4f20-928b-0e1c5100a431)      
![image](https://github.com/user-attachments/assets/20a788d3-ffe9-4ee8-951a-c87018bfccf9)   
![image](https://github.com/user-attachments/assets/f5c40524-c440-4bb9-980f-25bcbafe0264)





















