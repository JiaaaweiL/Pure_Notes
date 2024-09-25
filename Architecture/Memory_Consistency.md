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
