# 记载一点关于USB的东西。
![image](https://github.com/user-attachments/assets/a61b4da1-8b0a-497c-afb1-5008269e76c1)   

OTG/Host/Peripheral Core (Link)：    
这是主机（Host）、外设（Peripheral）或 OTG（On-The-Go）控制器的核心部分，它负责处理 USB 通信协议的高层逻辑，包括数据传输的调度、握手协议的实现、控制传输的管理等。这个核心部分主要是数字逻辑部分。

UTMI+ Link Core：这是负责处理物理层和链路层的核心。它包括了编码、解码以及从物理层接收或向物理层发送数据的功能。你教授提到你不需要处理模拟部分，所以你很可能是与这个部分打交道。

ULPI Link Wrapper：     
这个模块是一个“封装层”，它在 UTMI+ Link Core 和 ULPI 接口之间提供了接口桥接功能。ULPI 的主要目的是减少引脚数量，使得接口变得更加简化和标准化。所以，ULPI Link Wrapper 主要是将高层协议传输转换成适合通过 ULPI 接口传输的信号。

ULPI Interface：    
这个接口是 UTMI+ Link Core 和 ULPI PHY 之间的物理接口，用于在数字控制器和模拟收发器（PHY）之间传递信号。ULPI 接口通过降低引脚数简化了连接。   

ULPI PHY Wrapper：   
这是 ULPI PHY 的一部分，它起到了桥接和转换的作用。它将从 ULPI 接口接收的信号转换为可以由 UTMI+ PHY Core 理解的信号。这个模块和模拟电路紧密相关。   

UTMI+ PHY Core：   
这个核心负责处理物理层的操作，例如信号调节和恢复。它包括对差分信号的处理、编码和解码等。这个部分是直接与模拟电路相关的，你的教授提到的“模拟部分”可能就是指这部分。   






