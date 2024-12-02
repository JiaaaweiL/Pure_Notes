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


## Cache 的总体设计和考量：

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


### Virtual Memory
