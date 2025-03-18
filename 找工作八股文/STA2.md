## Timing
### delay
Delay between input change and output change. Delay is caused by capacitance and resistance in a circuit. 
![image](https://github.com/user-attachments/assets/3e1c0c46-67f7-40ab-ba54-41d1b4c5d7b9)   
Propagation delay: tpd = max delay from input to output.   
Contamination delay: tcd = min delay from input to output.   
![image](https://github.com/user-attachments/assets/ae83f98c-72f6-40bb-a1e0-67175c37c497)

### Glitch 
有的path快，有的path慢; When an input change causes an output to change multiple times after its contamination delay and before its propagation delay. 称之为毛刺。  
![image](https://github.com/user-attachments/assets/dd7369df-533f-4ecc-8fd7-8785ef39dd24)    
B从1到0， 先传播到下and, 造成了or门从1到0，然后在传播到了上and，造成输出从0到1，到or，Y = 1->0->1;  
**how to fix?** 
![image](https://github.com/user-attachments/assets/4e0321a2-4087-4fd0-b646-7c796f5822e6)   
讲人话就是 Kmap 全都连起来，不要有断层。Glitch的原因是卡诺图的两个group之间有gap。   
glitch在时序中不存在问题：只发生在contamination delay与propagation delay之间。 只要确保在采样过程中稳定即可避免。  

### Setup and hold time:  
Setup: time before clock edge that data must be stable.       
Hold time: time after clock edge data must be stable.      
aperture time: time around clock clock edge data must be stable. Ta = Tsetup + Thold       
![image](https://github.com/user-attachments/assets/251de0a1-d439-4591-a08b-fadf16888e55)       
Depends on the maximun delay from R1 though combinational logic to R2      
![image](https://github.com/user-attachments/assets/4a2d36a2-a3be-4157-b562-a9d5eb8675f6)        
Setup time 是由propagation delay决定的！     
Tpcq = T propagation to q， Tccq = T contamination to q    
![image](https://github.com/user-attachments/assets/2baf40e4-f80f-4b77-a94e-7fe95c10deae)      
一个例题     
![image](https://github.com/user-attachments/assets/71847c48-1b7c-4502-a562-c42c1c9ee07f)      
加上clock skew的情况： 
![image](https://github.com/user-attachments/assets/e5585f2c-b9a6-4892-834e-d33225341a4f)     
如果clock1 早，clock2晚，实际上，clock skew 带来了更多的余量 （Tc变相变长） 可比较坏的情况是：clk1 的时钟晚到了，所以Tc会被变相压短，所以有了公式：      
Tc > Tpcq + Tpd + Tsu + Tskew  => Tpd < Tc - Tpcq - Tskew - Tsu;     
![image](https://github.com/user-attachments/assets/4a847061-8112-4eef-b11f-9b9985cef0b6)   
正常情况下是Tccq + Tcd > T hold; 可是如果CLK2 晚到了，相当于CLK1 提早开始，所以就变成了Tccq + Tcd > T hold + T skew;    
变成了Tcd > T hold + Tskew - Tccq;   
 


