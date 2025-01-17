## Verilog/SV 的基础语法
### 1. Blocking, Non-Blocking的基础语法
" = " blocking 阻塞赋值是顺序执行的，即一行语句执行完后，下一行语句才会开始执行; 模拟逻辑电路中连续赋值, 用于组合电路
```systemverilog
    a = 1;       // 这行语句执行后，a的值变为1
    b = a + 1;   // 由于a已经被赋值为1，因此b的值为2
```

" <= " Non-Blocking 非阻塞赋值是并行执行的，也就是说，每一行赋值的右侧表达式会被立即计算，但赋值操作会在当前时间步结束时生效; 模拟寄存器的行为, 用于时序电路

```systemverilog
    a <= 1;       // a的值将在本次时钟周期结束后更新为1
    b <= a + 1;   // 此时a的值尚未更新，因此b的值是基于之前的a计算的
```

**case/casez/casex**
```systemverilog
case: 精确匹配
reg [1:0] sel;
always @(*) begin
    case (sel)
        2'b00: out = 1; // 精确匹配 sel == 2'b00
        2'b01: out = 2; // 精确匹配 sel == 2'b01
        default: out = 0; // 没有匹配到时的默认值
    endcase
end
```
```systemverilog
用?代替任意值，实现模糊匹配
reg [3:0] data;
always @(*) begin
    casez (data)
        4'b1???: out = 1; // 只关心最高位为1
        4'b01??: out = 2; // 只关心前两位为01
        default: out = 0; // 其他情况
    endcase
end

```
```systemverilog
casex 是最宽松的匹配方式，因为它会将所有的 x 和 z 位都视为通配符
reg [3:0] data;
always @(*) begin
    casex (data)
        4'b1xx?: out = 1; // 忽略 x 和 ?
        4'b01??: out = 2; // 忽略 x 和 ?
        default: out = 0;
    endcase
end

```

**>>和>>>**区别：>> 是逻辑右移，用0做padding， >>>是 算数右移，会以MSB做padding   
**== 和 === 的区别** ==用于比较值，不能比较x和z； ===可以用于比较x和z 但是===不能synthesis

```systemverilog
b = a + c;

使用了延迟控制，即 #delay_expression。
在这种情况下，b 的更新不会立即发生，而是会在指定的延迟时间后执行。
b = #delay_expression a + c;
```

### 2.boolean algebra 复习。再来找题吧

### 3. K-maps, 
隨便給⼀個truth table要可以⽤k map找出 product of sums 和 sum of products    
https://www.allaboutcircuits.com/textbook/digital/chpt-8/sum-product-notation/      
https://www.youtube.com/watch?v=eznPb3DWOQ0  

### 4. Comb Logic 2：1 mux -> 4:1 mux
![image](https://github.com/user-attachments/assets/13062c35-a29d-48b8-9bf8-08c55c261572)  
and/or/not/nand/nor/xor/xnor 用2：1 mux 拼  
AND:
![image](https://github.com/user-attachments/assets/ac8c76d1-0622-4d8d-b8c9-e7684ffe9539)
OR:
![image](https://github.com/user-attachments/assets/0517bdfa-f20d-4e1a-9585-aeaf52a0f6ba)
NOT: 
![image](https://github.com/user-attachments/assets/6c6868a4-e8b9-488d-ae3a-ec5296cea0d7)      
NAND: 一个AND + inverter   
![image](https://github.com/user-attachments/assets/83e118b3-6cca-43df-850c-5e9a3ede360b)  
NOR: 一个not + 一个or  
![image](https://github.com/user-attachments/assets/c6dfbd9c-c684-4b18-a1ea-ed2f6cf34f77)
XOR:   
![image](https://github.com/user-attachments/assets/351228aa-516a-454e-84fd-000c38e8647f)
XNOR:  
![image](https://github.com/user-attachments/assets/f12af4d0-f7ba-4a85-8905-66471d1b019d)

not/and/or/xor 用NAND拼      
NOT:
![image](https://github.com/user-attachments/assets/9a8a599e-1344-4bc6-9ff4-0dcd973d3bea)     
AND:  
![image](https://github.com/user-attachments/assets/8364cb11-4f2b-456a-a4ea-f77da34379d3)   
OR：   
![image](https://github.com/user-attachments/assets/df12e961-868c-4d6f-bc3e-10e23d02c117)   
XOR:
![image](https://github.com/user-attachments/assets/67b44757-fbcd-4cfd-b1d0-0ce2da73f289)  
![image](https://github.com/user-attachments/assets/60f06b49-c8a7-4dba-ace9-587485ab92e4)   


