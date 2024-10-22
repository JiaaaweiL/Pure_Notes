# Behavior

## set address
1. USB Host发送一个包裹给USB divice， 这个是一个SET-UP address
- 包裹的格式是： Sync  PID  ADDR  ENDP  CRC5  EOP
- **host 需要发送的数据包只有 PID(8bit， 正反PID） ADDR（7bit） + ENDP（4 bit） + CRC5(5 bit)** Sync + EOP(先不管) 应该是physical做的事情
- PID是 1101 0010  ADDR 的设置是 000 0000 endpoint的设置是 0000 CRC5交给python做，我不管


2. 第二部是送 control data packet。 这是一个config
- 包裹的格式是Sync  PID  Data	CRC16   EOP
- **host 需要发送的数据包只有PID(1Byte)， DATA(8byte) 跟CRC16(2byte)**
- PID是data0， 所以pid的值是0011 1100（表示data0） DATA的值BM-request type是（1byte 0x00）， bRequest是（0x05）， wValue是0x0001（2byte）	，wIndex 是0x0000， wLength是0x0000， CRC16 python自己管


3. 我们期望一个Ack从device到Host的。 
- 包裹的格式是：Sync	PID  EOP
- **PID的值是 0010 1101**
- **如果收到的值不是 0010 1101，就直接返回IDLE，所有重新跑 

4. 还要再从Host 到Device握手一次
- 包裹的格式是：Sync	PID  EOP
- **PID的值是 0010 1101**



## set config
**address 跟之前不一样**    
1. USB Host发送一个包裹给USB divice， 这个是一个SET-UP address
- 包裹的格式是： Sync  PID  ADDR  ENDP  CRC5  EOP
- **host 需要发送的数据包只有 PID(8bit， 正反PID） ADDR（7bit） + ENDP（4 bit） + CRC5(5 bit)** Sync + EOP(先不管) 应该是physical做的事情
- PID是 1101 0010  ADDR 的设置是 000 0001 endpoint的设置是 0000 CRC5交给python做，我不管

**这里不一样！！！**   
2. 第二部是送 control data packet。 这是一个config
- 包裹的格式是Sync  PID  Data	CRC16   EOP
- **host 需要发送的数据包只有PID(1Byte)， DATA(8byte) 跟CRC16(2byte)**
- PID是data0， 所以pid的值是0011 1100（表示data0） DATA的值BM-request type是（1byte 0x00）， bRequest是（0x09）**这里改了**， wValue是0x0001（2byte）	，wIndex 是0x0000， wLength是0x0000， CRC16 python自己管


3. 我们期望一个Ack从device到Host的。 
- 包裹的格式是：Sync	PID  EOP
- **PID的值是 0010 1101**
- **如果收到的值不是 0010 1101，就直接返回IDLE，所有重新跑 

4. 还要再从Host 到Device握手一次
- 包裹的格式是：Sync	PID  EOP
- **PID的值是 0010 1101**


## 之后是接受来自键盘的指令 做Interrupt transfer
1. 主机发送一个INTOKEN 包给device， 检查设备是否有值向传给主机
- 包裹的格式是： Sync  PID  ADDR  ENDP  CRC5  EOP
- **host 需要发送的数据包只有 PID(8bit， 正反PID） ADDR（7bit） + ENDP（4 bit） + CRC5(5 bit)** Sync + EOP(先不管) 应该是physical做的事情
- PID是 1001 0110(In token）  ADDR 的设置是 000 0001 endpoint的设置是 0001（**这里是1， 因为0是用来setup的**) CRC5交给python做，我不管

2. 从机向主机发送DATA包，
- 包裹的格式是： Sync  PID  DATA  CRC16  EOP
- **device需要发送的包裹只有PID， DATA 和CRC16** CRC16是python自动生成的，我不需要管
- PID是Data0 和Data1的**不确定** data 1 个byte大小， CRC 16 python自己生成

3. 主机向从机发送ACT包裹。 
- 包裹的格式是：Sync	PID  EOP
- **PID的值是 0010 1101**
- **如果收到的值不是 0010 1101，就直接返回IDLE，所有重新跑 

以此类推... 跑到断电








