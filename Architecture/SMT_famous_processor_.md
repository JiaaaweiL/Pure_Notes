#锐评SMT处理器
## Intel Pentium 4
![image](https://github.com/user-attachments/assets/f5f956e7-cfbc-4f02-bde5-f2326e0fc1fc)   

第一款商业化的通用SMT处理器（Intel的名字叫做Hyper-threading） 超标量，单核心，乱序，两个硬件上下文处理器。  
支持两种工作模式，单线程及多线程。   
资源划分是静态的，意味着，单线程情况下可以用所有的资源，而多线程情况下各用一半。坏处是浪费性能，好处是性能隔离好，单线程性能不受影响。    
Uop队列，指令队列，ROB都是静态划分的。physical architecture register是统一的。    
性能增加了15~27%在多线程和多任务场景下。 虽然SMT硬件开销小，但是资源共享和征用带来了设计复杂性。    

## IBM Power series


