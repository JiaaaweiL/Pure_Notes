## Sequence as an input: 
一直以来，输入是一个向量，输出是scalar or class（regression/classification）  
假设输入不是一个固定的向量，是一个set of vectors, 并且长度可变呢？  

- 问题1： 怎么把词汇表示成向量呢？
1. One hot encoding 一个很长的向量（多少个词，向量多长）  
2. word embedding 每一个词汇一个向量，有语义资讯  
![image](https://github.com/user-attachments/assets/3a2db8f7-13d3-4c00-aaa9-da215d8b8845)

还有什么是vector set as input的呢？ 声音信号！ 比如： 一串向量是25ms，一个stride是10ms 声音信号也是一堆向量  
graph也是一堆向量： 节点与节点之间的连接也是一堆向量的表示  

### 什么是输出呢？
1. 每一个向量有一个输出，输入输出的长度是一样的。eg： 词性标注，决定每一个词是名词/动词...
2. 只有一个输出：例如判断邮件是否是垃圾邮件
3. 机器自己决定要输出多少个label， sequence 2 sequence的任务

第一种任务： 本质上是Sequence labeling： 给Sequence里面的每一个向量一个label。  
### 直觉解法：
每一个向量输入到单独Fully Connected里面。每一个输入给一个输出。但是瑕疵非常大：没有上下文的联系了：  
词性标注例子： I saw a saw。 第一个saw是动词，第二个saw是锯子（名词）
解法2： Fully Connected之间有交叉，考虑一个window的上下文（相邻向量）：瑕疵是：能不能考虑整个sequence呢？  
  - window拉的超级大，盖住整个sequence 但问题一堆

## self-attention
一个self-attention 吃掉整个input，输出各个input with context。然后再由各个FC来处理。   
![image](https://github.com/user-attachments/assets/17207ab9-7021-4c6f-b9ae-25a84b39909e)
