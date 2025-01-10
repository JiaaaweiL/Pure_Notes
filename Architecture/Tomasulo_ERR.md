# 就来总结两种Renaming的方式。首先是Tomasulo
CC0:     
![image](https://github.com/user-attachments/assets/7499c671-16ef-4481-9c64-d24418d66335)        
CC1: 指令ADD F4, F2, F0 被放进了ROB 和 FP Adder1, RS标记清楚了operand和 ROB           
![image](https://github.com/user-attachments/assets/3d05bd6c-7e13-4108-b67d-06efa20a2c1c)      
CC2: 指令MULD F8, F4, F2被放进了ROB 和 FP MUL1, RS标清楚了operand来源是ROB0, 指令对应的是ROB1   
![image](https://github.com/user-attachments/assets/ac5d0140-6642-4d11-b871-8a66194d5a04)      
CC2: 指令ADD F6, F8, F0被放进了ROB和FP ADdder2， RS标清楚了operand来源是ROB1，指令对应的是ROB2    
![image](https://github.com/user-attachments/assets/4e64315b-37b6-47c6-98f5-f854fa08c2f8)    
CC3: 
