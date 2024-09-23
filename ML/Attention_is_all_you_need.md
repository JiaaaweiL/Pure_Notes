# 重点
### 简介
- 在主流的序列转录模型里面，主要依赖于循环或者CNN。一般是用encoder/decoder的架构。在性能最好的模型里面，会在encoder/decoder里面加入注意力机制  
- Transformer提供的贡献主要是： 提出了纯纯使用注意力机制的模型，抛弃了循环/卷积。模型简单，并且在两个机器翻译的branchmark中取得了领先。  
- 作者毒奶注意力机制可以被放在更多的任务上，不仅仅仅限于机器翻译。

### 结论：
- 把以前的循环层/encoder-decoder，全部换成了multi-head self-attention  
- 在机器翻译上，训练的更快，效果更好
- 对于纯注意力机制的模型感到有兴趣。此类方法可以被用在image/audio/video。好的泛化可能


# 以往的结构有什么问题？
### RNN网络
- 依赖时间顺序。给一个序列，序列的处理顺序是从左往右。对于t，需要计算hidden states H(t-1) 来学习历史信息（梯度爆炸？/前置信息容易丢失）。 要参考历史消息要提升ht，内存开销增大。
- 这是一个时序过程，RAW的dependency。导致任务难以并行。无法运用在SIMT上面
- attention再次之前已经用在了encoder-decoder上面。主要是负责encoder信息到decoder信息的有效传递
- Transformer 不再使用循环神经层。attention可以做更多并行。可以在短时间内达到更好结果

### 基于RNN的问题， 别人做了什么呢？
**用CNN来干掉RNN： 减少时序计算**
- 主要问题： 长序列难以建模。每一次一个神经元检测一个很小的窗口。如果序列长，需要非常多的层数去照顾两个间隔很远的像素。
  - 使用transformer的注意力机制就没有这个问题。一层能看到所有像素
  - 卷积可以做多个输出通道，attention没有，所以做了multi-head self-attention
    

# 模型
### 序列模型：编码器解码器
编码器：将一个序列输入（X1...Xn）编码成（Z1...Zn）输出。每一个Z是一个向量。    
解码器：拿到编码器的输出（作为输入），输出一个序列(Y1...Ym)    
这个模型的每一步是auto-regressive的过程（你的输入是自己的，上一个的输出） 过去时刻的输出是现在的输入。  
transformer使用了编码器和解码器的结构，叠加了self-attention和point-wise fully connected layer。  







