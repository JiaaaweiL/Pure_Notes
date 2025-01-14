### 1. functional coverage, code coverage 差异
这是对设计代码执行情况的统计分析。仿真工具可以自动生成代码覆盖率报告，评估测试用例是否遍历了代码中的所有语句、分支、条件等。代码覆盖率主要用于确保测试用例充分执行了设计的各个部分，但它无法直接反映设计功能的完整性。   

功能覆盖率：这是根据设计规范，由验证人员手动定义的覆盖率指标。功能覆盖率关注设计的功能点（features），评估测试用例是否覆盖了所有预期的功能场景。这需要验证人员根据功能设计文档指定验证计划，编写功能覆盖率模型（如 covergroup、coverpoint 等），并通过仿真收集数据。功能覆盖率独立于实际的设计代码或其结构，更加关注设计的功能需求是否被充分验证。   

### 2. assertion是什么
断言： 是用于验证设计行为的语句，旨在确保设计按预期工作。断言可在仿真期间动态检查，也可通过形式验证工具静态验证。
即时断言（Immediate Assertions）：这些断言是过程语句，主要用于仿真中即时检查特定条件。它们类似于 if 语句，但用于声明某个条件必须为真。如果条件为假，仿真器会生成错误消息。     
```systemverilog
assert (A == B) else $error("A 不等于 B");
```
并发断言（Concurrent Assertions）：这些断言用于检查随时间变化的设计行为，通常与时钟相关联。它们用于验证特定的时序关系或协议。   
```systemverilog
assert property (@(posedge clk) req |-> ##[1:2] ack);
```
此断言表示：在时钟上升沿，如果 req 为真，则在接下来的 1 到 2 个时钟周期内，ack 必须为真。
验证设计行为：确保设计在各种条件下都能按预期运行。   
提供功能覆盖信息：帮助衡量测试用例的有效性，确保所有预期的设计行为都被测试到。    
辅助调试：在仿真过程中，当断言失败时，提供有价值的调试信息，帮助快速定位问题。    

### 3. Constraint random, direct test 差在哪？
![image](https://github.com/user-attachments/assets/c432efe8-d014-473b-993e-1ffa6e7c340e)



### 4. Fork join, join_any, join_none 
![image](https://github.com/user-attachments/assets/a5dcae6e-7662-4636-85eb-4c192c8d3368)      
一道fork join的经典面试题：   
现在有定义好的三个子线程do1,do2,do3，在task中并行运行这三个子线程，其中只要有任何一个线程结束，都退出并行运行块，并打印DONE。要求分别用fork-join、fork-join_any，fork-join_none来实现   
```systemverilog
//用fork  join实现
task test();
  fork : tag
    begin
      sub1();
      disable tag;
    end
    begin
      sub2();
      disable tag;
    end
    begin
      sub3();
      disable tag;
    end
  join
  $display("done");
endtask : test
//fork:tag 定义了一个并行运行块，并给它加了一个名字 tag
//在任意一个线程完成时，执行 disable tag，立刻终止整个并行块 tag，从而停止其他线程
//用fork  join_any实现
task test();
  fork:tag
    sub1();
    sub2();
    sub3();
  join_any
  disable tag;
  $display("done");
endtask : test
 
//用fork  join_none实现
task test();
  event e;
  fork : tag
    begin
      sub1();
      -> e;
    end
    begin
      sub2();
      -> e;
    end
    begin
      sub3();
      -> e;
    end
  join_none
  @ e;
  disable tag;
  $display("done");
endtask : test
//join_none 表示立即返回，而不会等待任何线程完成。
//使用 event e 和 -> e 机制：在线程完成时，触发事件 e。
//主线程通过 @ e 语句等待某个线程触发事件。
//一旦某个线程触发了事件 e，主线程执行 disable tag，中止其余线程。
```

### 5. 写Constraint的题目
1. constraint a var like randc without using randc keyword   
首先，啥是rand，啥是randc 还有$urandom, $urandom_range, $random   
rand 和randc 可以作为类内随机数。 rand 和 randc的区别就是rand是纯纯随机的，它不保证生成的值是非重复的，也不保证遍历所有可能值。randc每次调用 randomize()，randc 变量会生成一个新值，但不会重复之前生成过的值。当所有可能值都被生成后，它会重新开始遍历。保证遍历所有可能值，并且在每次循环内无重复。   
如何不用randc写出一个像randc的随机数呢？用一个池子，放入所有数。每次从池里取值   
```systemverilog
class abc;
  rand logic[2:0] a;
  int saved[$];
  
  constraint c1 { !(a inside saved);}
  
  function void post_randomize();
    saved.push_back(a);
    if(saved.size == 2**3)begin
      saved = {};
    end 
  endfunction
  
endclass

module abc;
  abc a1;
  
  initial begin
    a1 = new();
    repeat(8)begin
    a1.randomize();
    $display(a1.a);
    end
  end 
endmodule
```


2. Constraint a var like unique without using unique keyword
Unique的两种场景：  确保分支互斥 当 unique 关键字用于条件语句时，表示编译器/模拟器保证每个条件分支的条件互斥，即在任意时刻只有一个分支条件为真。如果多个分支条件可能同时为真，编译器会给出警告或报错。经典案例是Unique Case 只确保有一个case的分支是工作的，不能有两个case的分支同时工作
```systemverilog
class UniqueConstraintArray;
  rand int values[3]; // 定义一个数组，包含3个随机变量

  // 约束数组中所有元素互不相同
  constraint unique_values {
    foreach (values[i]) {
      foreach (values[j]) {
        if (i != j) values[i] != values[j];
      }
    }
  }

  // 显示生成的值
  function void display();
    $display("Values = %p", values);
  endfunction
endclass
```

3.  Constraint to check if a given sudoku is valid or not 约束解数独。这玩意太抽象了

   
5.  Constraint a 16 bit var has 2 bit '1'
```systemverilog
class TwoBitOnesConstraint;
    rand bit [15:0] var; // 16位随机变量

    // 约束：确保16位变量中恰好有2个'1'
    constraint two_bit_ones {
        $countones(var) == 2;
    }
    // 打印结果
    function void display();
        $display("var = %b (decimal: %0d)", var, var);
    endfunction
endclass
```

6. constraint 2 var that has 1 bit flipped 0 ->1
```systemverilog
 constraint one_bit_flip {
        b & ~a == (b ^ a);          // b 中新增的位是翻转位
        $countones(b & ~a) == 1;    // b 中比 a 多的位只有一个
    }
```
b & ~a == (b ^ a)： 明确规定了 所有差异只能是 0 -> 1，而不能是其他变化（如 1 -> 0 或无效的状态）。       
$countones(b & ~a) == 1：限制新增的 1 的个数（从 0 -> 1）。     


### 6. FIFO 验证思路
同時讀寫，讀寫數據正確性檢查       
FIFO滿標誌位檢查      
FIFO空標誌位檢查          
寫過程中發生寫復位，寫數據和FIFO滿標誌位被清空       
讀過程中發生讀復位，讀數據和讀空標誌位被清空     
讀寫時鐘相位相同，異步FIFO能正常工作  
讀寫時鐘相位不同，異步FIFO能正常工作  
寫時鐘等於讀時鐘，異步FIFO能正常工作  
寫時鐘快於讀時鐘，異步FIFO能正常工作  
寫時鐘慢於讀時鐘，異步FIFO能正常工作  
寫過程中發生寫復位以後，異步FIFO能繼續正常工作  
讀過程中發生讀復位以後，異步FIFO能繼續正常工作  
对于FIFO的满的验证，需要考虑如果设计用满作反压，那么就是要求，FULL有效后，立即停止发包，因此Drive必须要要有这种的功能。同时也需要考虑异常场景，即FIFO满后，仍然往FIFO发包，FIFO正常丢包，不挂死，当FIFO 满撤销后，还能正常继续工作。   

### 7. Round robin arbiter 验证思路   
功能正确： 按照RR的逻辑轮询分配资源
  - 单一请求者的情况下，是否正确分配
  - 多个请求者的情况下，是否按顺序分配
  - 无请求者的情况下，valid是否为低
  - 全部请求的情况下，是否按照正确的顺序分配
公平性：所有请求者都能按照顺序被仲裁到    
边界条件：无请求或者全部请求的情况下行为正确
  - 复位测试
  - 连续请求
  - 指针循环一遍，是否恢复到第一个请求者
异常场景： 极端负载，复位或者错误的输入条件下能正确的工作
  - 无效输入的i情况下
  - 高负载的情况下
  - 连续请求的情况下

### 8. 有一个module有三个input，分别代表三个点。每个input是X, Y的坐标。当三点共线时，output1。 请问要如何验证？
思路是 两个点算出y = a*x + b  带入第三个点的x算出y坐标，然后在和第三个点的y的原始坐标比较。如果一样则证明共线。  
还需要处理conner case： 垂直然后斜率无穷大。浮点数可能导致的误差（给一个误差范围， 然后用double 而不是普通float）。三个点都是同一个点，也理应共线。   

### 9. 有一个memory entry(16 bit)， 存在两个bug， 第一个bug是有些bit会卡在1/0不动，我们需要那些write/read 才能找出该bit？ 第二个bug是有两个bit会互相的swap，要write/read哪些test vector才能找出这两个bit？  
Bug1：
1. 对memory写入0和1来对比结果。 如果某个位写入1后仍然显示0，则代表该位卡在了0  
2. 对memory写入0之后仍为1， 则说明该位卡在1  
Bug2:    
通过设置单个位为 1，其他位为 0 的形式，观察读取结果中哪个 bit 被置位。     
每次写入单个位为 1，并读取 memory：    
如果读取值中出现两个 bit 为 1，说明发生了 bit-swap。      
如果读取值中 1 的位置与写入时不同，也说明发生了 bit-swap。  
