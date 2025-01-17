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


https://www.allaboutcircuits.com/textbook/digital/chpt-8/sum-product-notation/
https://www.youtube.com/watch?v=eznPb3DWOQ0
