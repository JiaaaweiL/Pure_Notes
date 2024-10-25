# USB_ULPI_behavior

## Step1: Connect device
![image](https://github.com/user-attachments/assets/e548419f-5d67-483f-b5f4-26414b814cc4)  
键盘插上USB设备之后，做的第一件事情就是connect device   
首先把dir 设置成1， nxt保持0；   
发送的package包含linestate（01）   
[7]   alt_int  = 0   [6]   ID= 0  （假设没有接入OTG设备）   [5:4] RxEvent  = 01   [3:2] VbusState = 10     [1:0] LineState = 01        
8’b = 00011001；    
然后一个cc之后，dir降成0；net保持0；    

## Step2：Setup address
然后开始setup address；这个阶段是Host发东西给device，ULPI那一侧只需要接收东西。      
首先发第一个包裹是PID；  然后是ADDR+ENP +CRC5    
为了能够接收到发送的包裹，   
![image](https://github.com/user-attachments/assets/061d4a26-d081-47c5-bbab-cc1d1725f750)   
首先先期望接收带PID的TX CMD. dir = 0; **nxt = 1，表示PHY已经准备好接收包裹。**  一直接收，直到遇到了stp，然后NXT置0；   

然后开始接收ctrl data。同样的道理， 首先先期望接收带PID的TX CMD. dir = 0; 收到TX CMD之后 nxt置位1 **nxt = 1，表示PHY已经准备好接收包裹。**  一直接收，直到遇到了stp，然后NXT置0；    
![image](https://github.com/user-attachments/assets/14629dfd-5619-4ef1-8c5e-35d65d910d46)   

发送完包裹之后，主机期望device发送一个receive的ACK给从机，     
![image](https://github.com/user-attachments/assets/353221ff-7614-4b8b-a4a5-dfa3bdcb3f97)      
第一拍 DIR置1，Nxt置位0，      
第二拍子 NXT置位1， Data发送一个ACK包，     
第三拍子 NXT置位0， Dir置位0，     

然后主机还要反过去握手一次。     
第一拍子dir = 0; 可以直接发送       
探测到之后 NXT拉高 第二排接收TX CMD        
第二拍子发一个PID    
第三拍子发一个STP， 然后NXT和STP都拉低。    

## Step3： Set up Config ULPI的东西是一样的，步骤也是一样的，只是config的值不一样。

## Step4： IN_TOKEN并且interrupt数据   

主机先发一个IN_TOKEN  报文格式是Sync PID ADDR ENDP CRC5 EOP   
这里的第一拍先是nxt为0 dir是0，然后发TX_CMD  
第二排nxt拉高接受CMD，然后发data（addr endp 和crc5）   
发完之后的某一拍STP拉高，完了STP拉低，后nxt也拉低。

然后主机期望从机发消息给主机。这里是发一个标准的DATA包+RX_CMD 包裹。      
RX_CMD发送的时候NXT为低（告知这是一个cmd而不是data  5：4位是00，告知是EOP）         
然后结束 turn around    

上述顺序以此类推   

## Step5： 关机。
找个时间，发一个包，device发给host
nxt是0， dir是1， 包的内容是RX_CMD ,标注清楚 [1：0]是 00， [5：4]是10就完事了
