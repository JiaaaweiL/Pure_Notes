# 来一点儿纯的Memory Consistency
### Memory Consistency 是啥玩意？ 
定义是：Consistency 定义了load/store的规则，和这些操作什么时候施加于内存。Consistency的责任是确保共享内存的行为正确性。   **Consistency models define correct shared memory behavior in terms of loads and stores (memory 
reads and writes), without reference to caches or coherence.**  
Consistency model 定义这是否是一个正确的行为（用户是否需要采取其他行为去得到正确结果）或不正确（系统需要先
进行排序）  

## Share Memory 会带来什么问题？
考虑这样一份代码 （假设所有寄存器的初始值是0）
![image](https://github.com/user-attachments/assets/01961005-b43a-4590-9057-17cede3dcfe3)   
大部分程序员期望Core2的R2值是NEW。 然而，R2的值在现代计算机系统里面可以是0。  
为什么呢？ 这是因为现代计算机系统可以重新排序C1的s1和s2。这个重排看上去是正确的（S1,S2没有read after write 
dependency） 然后结果就变成了**S2, L1, L2, S1** 然后r2的结果就可以成为0。

再来看一个案例：
![image](https://github.com/user-attachments/assets/d02e3494-2215-43ef-a05c-fe72641fff36)
理论上来说，三种结果。(0, NEW), (NEW, 0), (NEW, NEW)  但是在X86中， R1, R2的值可以是(0，0)， 因为X86中会
用FIFO写缓冲来提升性能。也就是说，L1, L2可能会被先发射出去。 因为多处理器默认都是非确定性的，所以定义共享内存
行为时，我们必须考虑非确定性。  

内存一致性模型，或者更简单地说，内存模型，是使用共享内存执行的多线程程序允许的行为的规范。对于使用特定输入数据
执行的多线程程序，它指定动态加载可能返回的值以及内存的最终状态。  总的来说，MC是给定一个规则，将程序分为遵守MC或者不遵守MC

# 最简单的， SC sequential consistency
Single Core的consistency： 执行结果与操作按照程序指定的顺序执行的结果相同  
Multi Core的consistency： 执行的结果与 将几个线程的程序按照顺序排在一起，且线程内的命令顺序与程序顺序一致 相同   

**来点儿数学表达**
![image](https://github.com/user-attachments/assets/eb1fcc32-6288-4d48-8311-8f106a2bfdcb)  
La < Lb : La先执行，Lb后执行。p: program, m: memory  
![image](https://github.com/user-attachments/assets/091991e9-9a01-4a56-89af-b8550fc19532)  
意味着 La 永远取最后一次store的值  

### 如果是：implemetation 怎么做呢？
单核： 单核都不需要什么fansy skill, 只需要保证当context switch的时候，所有延迟的内存操作必须施加在内存上即可。  
多核： 简单的是switch 轮到谁了就是谁，用时间做切分，随机也可以。  
![image](https://github.com/user-attachments/assets/22817e30-6850-4769-87af-39fdd4a71b6f)  

**SC 永远是golden rule**

### SC implementation 和CC的优化
- Non-Binding Prefetching
先来介绍介绍这是啥：  （by GPT）  
普通预取：会实际把数据从内存加载到缓存，可能导致缓存污染。如果数据被使用，它可以加速程序执行；如果不被使用，可能浪费缓存空间。    
非绑定预取：不会直接加载数据，而是提前准备缓存的一致性权限（如“共享”或“已修改”状态），并且对程序行为不产生强制影响。如果预取的数据没有被使用，程序执行不会受到影响。    
不强制绑定：非绑定预取不会改变寄存器或数据，仅仅是改变缓存中的一致性状态。也就是说，它不会对程序的执行逻辑产生直接影响。如果某个预取失败（即预取的数据没有被后续使用），程序的执行不会因此出错或异常。  
**意义点在于：** 核心变多之后，修改一致性状态需要与其他核心竞争/通信。Non-Binding Prefetching 可以隐藏这部分的延迟  
**如何恢复？** 不需要恢复，错了就错了？不行啊！ 一个invalid 的cache line被错误的non-Binding prefetch，只修改了状态位的情况下被证明是错误的prefetch，在不更新data然后写回内存的时候会错误。怎么改？ （自己猜的，加一个bit证明这个玩意被真正的fetch过data，否则不写回内存）    
Non-binding Prefetch 不会改变MC状态  

- Speculative Core （这里不是OoO的CPU）
如果运用了Dynamically Scheduled Cores则会出现问题：（这里的Speculative Core只speculative了branch的部分，load/store的顺序还是遵循源程序顺序）
Speculative本质上不会出问题（违反SC）， 原因很简单，因为squashed 内存指令看起来就像non-binding prefetching。  
举例说明：  
1. Load指令mispredict， Core丢掉寄存器的值，把流水线清空，完事。 因为是load，non-binding prefetch做的invalid/modified 变成share也不需要考虑别的因素，缓存被换掉的时候反正不会写回内存的   
2. Store指令会先做non-binding prefetch，然后等它百分之一百会被commit的时候再施加于内存。（回想一下pipeline，branch是在EX被确定的，然后会把流水线flush掉。根本不存在让错误的LS走到第四个stage）     

- Dynamically Scheduled Core（没错，这个core是OoO的core了）   
会有一个核心问题：Memory Consistency Speculation，考虑两个Load指令，如果被reorder了，当然会违反SC。  
解决思路是： Speculating on SC 需要核去证明你的speculation是对的。  两种方法：
1. 检测并且确保在commit L2之前，对于缓存的更改局限在私有缓存范围内。换句话说： 在私有内存里面进行的操作不会被别的核心看见，可以放心操作
2. 在commit的时候进行重播load操作。如果重播数据没变代表这是有效且未更改的（很蠢，而且貌似仅仅只针对load。没办法重播store）  

- Non-Binding Prefetching in Dynamically Scheduled Core
  最主要的是： 从全局系统的角度看，所有核心的加载和存储操作必须按照一定的顺序执行。无论这些操作是否在私有缓存中乱序执行，最终它们被提交到共享内存或主存时，必须看起来像是按照程序顺序执行的。

**讲白了，想怎么存都可以，提交顺序一定是按照程序的顺序！！！**

### 原子操作对于SC的意义：没有atomic operation就没有multi-threading
核心是：确保RMW(read modified write)中间的没有任何指令的插入。  
- 简单的方法：锁总线！这样就不会有指令打断RMW了
- 稍微复杂一点的方法： 对于需要修改的锁，将其的CC状态锁起来，例如，改成Modified，然后未完成之前不许其他缓存块访问此块
 
### 总结起来的Case study： MIPS R10K
4-way superscalar RISC processor core,有分支预测单元。 用directory coherence protocal。  
运行时，R10K 以程序顺序发射LS指令到地址queue。load-store中间有一个forwarding。地址相同的情况下，load指令的data可以直接从最老store中获取。 如果上述和不成立，LS指令的commit顺序是程序顺序，然后从queue中移除地址。Store指令的操作顺序是：L1cache 保持M state（用或者不用non-binding prefetch无所谓）， 然后store将数据写入cache的同时commit掉指令。

# TSO x86的memory model  
为什么有TSO? 因为write buffer实在是太好用了。 Store指令可以直接将store value放进store buffer然后继续执行，减少对CPU的阻塞，隐藏了延迟。  
对于单核处理器而言，write buffer基本上是架构不可见的。 但是对于多核处理器而言，write buffer的架构不可见策略就完全错了。  
   ![image](https://github.com/user-attachments/assets/d5422a40-d796-4dc6-aabd-3d7157fc6bfd)   
我们期望的值有什么？ (0, new), (new, 0), (new, new) 然而实际上的可能是，出现的结果可以是（0, 0）,这是因为写操作被丢尽写缓存里面。  

### SC 和 TSO 有什么不同？
SC 强调的四种关系 load -> load,    load -> store,   Store -> Store,   Store -> Load, 然而TSO 只强调前三种，剔除了最后一种关系。  
数学表达
![image](https://github.com/user-attachments/assets/7f69f5e5-d77d-4347-97e5-79aa32d525c1)   
说白了，就是甩锅给程序员。要么你自己插fance，要么就别期望两个store-load施加到内存的顺序有保证
Fance的数学表达
![image](https://github.com/user-attachments/assets/ca23b414-f92a-494a-8591-8bd75f25051e)  
将白了。Fance会保证fance之前的内存操作都被施加于内存。之后才能跨过fance进行fance之后的操作

### Implementing TSO
TSO的Implementation和SC的其实很像， 多了一个per-core FIFO write buffers. 
TSO SC的主要区别是TSO允许存储操作可以延迟可见。换句话说，存储操作不需要立即对其他核心或线程可见，而是可以先进入一个写缓冲区，核心继续执行后续指令。这种机制通过引入每个核心的FIFO写缓冲区来实现  
TSO允许bypass。有load先去store queue里面找，看看能不能抄到答案  
写缓冲区的引入使得存储操作可以被延迟，但这并不会破坏MC，因为从全局角度看，内存操作的commit必须顺序发生。  
多线程中，TSO write buffer 必须对于每一个线程的上下文私有。因此，必须是（线程切换前清空/associative with threadID)    

### Atomic Instruction
和SC的Atomic Instruction基本相同。唯一的不同是需要支持write buffer。有可能出现的情况是要Write may be written to the write buffer.   
**因此就有如下两种顺序要求**
加载不能越过加载：由于TSO模型要求程序顺序的加载指令必须按顺序执行，RMW中的加载部分不能在之前的加载指令之前执行。  
存储不能越过存储：TSO 还规定存储指令不能乱序，因此RMW 中的存储部分不能在之前的存储指令之前执行。包括在写缓冲区中的待写。  
而且，因为是atomic operation，所以意味着，RMW的read（load) 和 write（store)得是一体的。  
所以，RMW之前的store必须首先被提交（store不能超过store） -> **清空缓冲区**   
因为RMW是原子操作，所以**必须提前抢占M**（意味着只有原子操作能够修改对应位置缓存块）并且在操作期间**不 能 放 弃 M state** 意味着需要锁住这个缓存状态位   
**一种优化方法：**  
假设写缓冲区有这个你需要原子操作的地址，做法是可以不用清空缓冲区+无需提前抢占M state。原因如下：
因为写缓冲区有，所以可以直接bypass的读到，完成了read（load） 完了因为已经在写缓冲区了，意味着M state的权限已经归于这个核心了，所以也不需要抢占，只需要保持。然后仅仅需要将新的写结果放入queue的新的entry就可以了  

### Fance的Implementation
因为使用频率非常低，所以非常简单粗暴的方法也不会带来很多性能损耗，那就是：在FANCE指令发出时候，清空读写queue，然后才允许新的指令进来。    

### 评价
好的内存一致性模型应该具备的四个关键要素（4P）Programmability（可编程性）， Performance（性能）， Portability（可移植性）， Precision（精确性）   
SC最直观简单，TSO相对来说难理解一点。理论上TSO会效果更好，但是因为有speculative execution，SC的性能也不会差太多。两者的定义都很percise  

# Relaxed Memory Consistency
Relax memory consistency model 追求值保留那些程序所需要的必要的顺序。 因为只最小化了保序要求，理论上能更灵活获取更多的性能 

![image](https://github.com/user-attachments/assets/9a5f24bf-7086-4a46-99e5-d6941be8bba4)  
考虑一下图上的例子。 理论上来说，我们期望的值是R2, R3 都是New。   
这里面必须的MC顺序是
S1 → S3 → L1 loads SET → L2  和   S2 → S3 → L1 loads SET → L3 然而，对于SC和TSO而言，不必要但是被执行的MC 顺序是S1 -> S2, 和L2 -> L3。  保存这些额外的顺序将会对优化和性能造成损害。这些东西其实是不必要的。
再来看一个顺序  
 ![image](https://github.com/user-attachments/assets/c9c6fbf2-b21a-44a5-a8da-bc887b4f3fc5)   
 在这个例子里面的保序模型是 All L1i, All S1j → R1 → A2 → All L2i, All S2j, 仔细读来就会发现其中包含了很多没有用的order。正确方法是：如果两个critical region中包含
 了相同的地址（对于相同地址进行load，store） 则需要注意MC。 那也就是：    
All L1i and S1j can be in any order with respect to each other, and   
All L2i and S2j can be in any order with respect to each other.   
意味着：critical region中的Load Store，如果地址不同则无需保序。但是两个CR要互相保序。  


### 一些可能的优化：
non-fifo， coalescing write buffer：  这里的coalescing的意思是： 当两个写操作目标地址一致的时候，可以统一写操作，避免多次写入。 很显然是违反TSO的，因为TSO强制要求保序（program order） relax model可以这么做，如果没有fence强制隔开的化。   

简单的对core speculation的支持： 在强一致性模型中，也可以在commit之前做乱序做推测执行。例子是R10K 处理器。 但是要具备相应的检查机制（参照SC模型的机制1和2，保持修改缓存的私有和重放执行） 代价显而易见，是硬件复杂度和面积功耗。 另外还有弱一致性，完全随意乱序， 并且不需要一致性检查。   

coherence和consistency的耦合/解耦: 上述所有讨论基于的模型是SWMR(single writer multiple reader) 如果放开这个原则可以提高性能，但它引入了很多复杂性，尤其是在验证正确性时。如果允许部分核心看到不同版本的数据，这将大大增加系统设计中的逻辑复杂性，因为需要设计相应的机制来追踪和解决这些不一致。  

## XC模型（这就是个 teaching purposes 模型！大概率是不存在的）
提供了什么：Fence。当顺序需要的时候，会用fence保证。除此之外，load 和 store 都是乱序的。   
Notation: core Ci 处理load/store指令（Xi），然后fence指令，然后load/store指令（Yi）。FENCE起到了一个分隔操作的作用，确保前后的内存操作不会乱序执行。两个Fance指令之间互相保序，但是fance指令作用只能在一个核内。   
所以XC的memory order就会是：  
Load → FENCE        Store → FENCE        
FENCE → FENCE        FENCE → Load         FENCE → Store
对于同地址的Load/Store， XC将会以TSO来保序，例如  
![image](https://github.com/user-attachments/assets/a7b40984-94f5-4ed7-a2b6-857b49445840)    
数学表示也相当简单，跟之前的一样，即：  
有fance存在，就必须把fance之前的内存操作清空，然后才能是fance。   
fance之间必须互相保序。    
同地址的load store操作，需要遵循TSO（是TSO噢不是SC）   
![image](https://github.com/user-attachments/assets/f9011d6b-4eac-44b6-93f9-8979ce5b0a06)  




