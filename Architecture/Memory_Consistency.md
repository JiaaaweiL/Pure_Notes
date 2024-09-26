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

## 最简单的， SC sequential consistency
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

## TSO x86的memory model  



