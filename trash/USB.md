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


[PROTOCAL]https://www.beyondlogic.org/usbnutshell/usb1.shtml    
[ULPI interface]https://www.sparkfun.com/datasheets/Components/SMD/ULPI_v1_1.pdf

## ULPI
一堆接口， 1bit 的clock， 8bit的data， 1bit的dir， 1bit的stp， 1bit的nxt；      
data是双向接口，dir是phy控制的单向接口，dir = 0表示数据从链路到phy，dir = 1则表示数据从phy到链路。     
stp信号是单向，用于表示数据包传输结束，nxt信号表示下一字节是否准备好传输。   
ULPI 定义了两种命令字节：    
Transmit Command Byte (TX CMD)：链路发起的命令，传输到 PHY。   
Receive Command Byte (RX CMD)：PHY 发起的命令，接收自链路。（我们想要键盘写入内存，所以只照顾PHY发起的命令）    
![image](https://github.com/user-attachments/assets/db15a330-d7d0-4d6d-be82-fe734e86a01a)     
什么时候发送RX? 参照下图。 nxt 和stp都是0，因为是CMD命令。   
![image](https://github.com/user-attachments/assets/49962bd7-4cde-4f42-80f2-80bcc4883902)    

![image](https://github.com/user-attachments/assets/b58f1f43-ace8-4985-a456-77b38d2ff833)



LineState 的值代表了 USB D+ 和 D- 信号线的状态，它可以帮助判断 USB 设备当前的连接和通信状态。LineState 的具体值取决于设备所处的状态，例如：是否有设备连接、是否正在传输数据、处于全速（Full-Speed）或高速（High-Speed）模式等。

根据 USB 的标准，LineState（LineState(1:0)）的值通常代表以下几种情况：

LineState 可能的值和含义：
00: SE0（Single Ended 0）

解释：表示数据线处于 SE0 状态，通常意味着 USB 设备断开连接，或 USB 设备正在发送信号结束（EOP, End of Packet）。也可以表示复位状态。
常见场景：设备未连接或复位。
01: J-state

解释：表示数据线处于 J 状态。这代表设备处于空闲状态或者数据线处于 Full-Speed 模式下的空闲状态（D+ 为高电平，D- 为低电平）。
常见场景：在 Full-Speed 设备中，设备空闲时通常表现为 J-state。
10: K-state

解释：表示数据线处于 K 状态，通常是 Full-Speed 模式下的 USB 信号切换状态（D+ 为低电平，D- 为高电平）。在高速（High-Speed）模式下，K 状态表示信号是 High-Speed Chirp 信号的一部分。
常见场景：在数据传输或高速模式下的信号状态。
11: 未定义（通常不会出现）

解释：在大多数情况下，11 是无效或未定义的状态。在实际实现中，11 不应出现。
哪个值最合理？
具体哪个 LineState 最合理，取决于你当前的 USB 设备状态：

设备未连接或复位：最常见的 LineState 是 00（SE0），表示设备未连接或总线处于复位状态。

设备处于空闲状态（未传输数据）：Full-Speed 模式下，LineState 通常是 01（J-state），表示设备已经连接但没有传输数据。

设备正在发送数据：当设备正在传输数据时，LineState 会在 01（J-state） 和 10（K-state） 之间切换。

高速模式下的信号握手：如果设备处于 High-Speed 模式，LineState 可能会进入 K-state，即 10，用于发送信号握手或 chirp。

总结
如果你的设备处于空闲状态（没有数据传输），那么 01（J-state） 是最合理的。
如果设备未连接或复位，则 00（SE0） 是最常见的状态。
如果设备正在进行数据传输，LineState 会在 01（J-state） 和 10（K-state） 之间切换。
为了确保你的 ulpi_wrapper 正确处理 LineState 信号，你需要根据不同的设备状态设计测试用例，并验证 LineState(1:0) 是否符合预期的 USB 设备状态。

LineState（data[1:0]）
描述：UTMI+ 线路状态信号，用于指示 USB D+ 和 D- 信号线的状态。这些信号线表示 USB 设备当前的逻辑电平状态，通过该状态可以判断 USB 设备的连接和通信状态。
data[0] = LineState(0)：表示线路状态的第 0 位。
data[1] = LineState(1)：表示线路状态的第 1 位。
通过 LineState(0) 和 LineState(1)，链路可以判断 USB 设备的当前通信状态。通常，USB 信号线的状态变化会对应于设备连接、断开、数据传输等操作。

2. Vbus State（data[3:2]）
描述：用于指示 VBUS 电压状态的信号。VBUS 是 USB 设备供电的电压线路，这些状态信号可以帮助链路判断 USB 设备的电源连接情况，特别是在 OTG 模式下。

00：VBUS < VB_SESS_END 表示 VBUS 电压低于 VB_SESS_END，即没有足够的电压提供给设备，通常表示设备未插入或电源断开。

01：VB_SESS_END < VBUS < VSESS_VLD 表示 VBUS 电压高于 session 结束电压，但低于 session 有效电压。设备可能处于某种非活动状态。

10：VSESS_VLD < VBUS < VA_VBUS_VLD 表示 VBUS 电压处于有效电压范围内，表示 USB 设备处于活动状态或即将进入活动状态。

11：VA_VBUS_VLD < VBUS 表示 VBUS 电压高于 VA_VBUS_VLD，设备处于完全活动状态，通常表示设备已插入且电源充足。

通过这些电压状态，链路可以确定 USB 设备的插入状态、断开状态或电源状态。

3. RxEvent（data[5:4]）
描述：表示编码的 UTMI 事件信号，这些信号用于指示 USB 接收过程中的事件情况。

RxEvent 值	RxActive	RxError	HostDisconnect
00	0	0	0
01	1	0	0
11	1	1	0
X0	X	X	1
RxActive：当 RxActive 为 1 时，表示 USB 正在接收数据包。此时数据传输正在进行中。

RxError：当 RxError 为 1 时，表示接收过程中出现错误，数据包可能存在问题。

HostDisconnect：当 HostDisconnect 为 1 时，表示 USB 主机已断开，设备可能已拔出或不再通信。

这些事件信号帮助链路判断接收过程中的状态，特别是 USB 接收是否正常、是否有错误发生以及设备是否断开。

4. ID（data[6]）
描述：该信号与 IdGnd 信号相关联（对应于 UTMI+ 的 IdDig 信号）。该信号用于 OTG（On-The-Go）模式下，用来判断 USB 设备是处于主机模式还是设备模式。

有效性：IdPullup 设置为 1 后 50 毫秒内该信号有效。通过检测 ID 信号，链路可以确定设备当前的角色：

IdGnd（ID = 0）：设备处于主机模式。
IdDig（ID = 1）：设备处于从设备模式。
5. alt_int（data[7]）
描述：该信号表示 非 USB 中断 的发生。当发生非 USB 中断时，PHY 会将该位置为 1。具体的中断源可以通过读取 Carkit Interrupt Latch 寄存器来确定。

例如，可能是来自于 Carkit 或其他设备的非标准 USB 中断。当该信号为高时，链路需要进一步检查中断寄存器，以确定中断的具体原因。

总结：
LineState：反映 USB 数据线 D+/D- 的状态，指示 USB 设备的连接和通信情况。
Vbus State：反映 USB 电源线 VBUS 的电压状态，用于判断设备的供电和插入状态。
RxEvent：指示接收过程中是否有数据传输、错误或者主机断开等事件发生。
ID：在 OTG 模式下，用于指示 USB 设备是主机模式还是设备模式。
alt_int：指示非 USB 中断的发生，需要进一步读取中断寄存器来确定中断的来源。
你可以根据这些信号的含义来设计验证方案，确保 ulpi_wrapper 对这些状态信号的处理符合规范。
