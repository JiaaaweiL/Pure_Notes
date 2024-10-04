# 验一个小小的ALU.sv

以下是coverage.svh代码

```systemverilog
covergroup cg with function sample(
  bit [63:0] a,
  bit [63:0] b,
  bit [3:0]  op,
  bit [5:0]  b_shift_bits
);
  // This generates a bunch of ranges across possible values of a,
  // and ensures at least one value in each range is covered.
  coverpoint a;

  // Same for b.
  coverpoint b;

  // Checks that your testbench exercises all valid operations.
  coverpoint op {
    bins range[] = {[0:8]};
  }

  // This is called a "cross cover" -- it checks that for both
  // left and right shifts, you test all possible values of b
  // (there are only 64 possibilities for the shift value).

  all_shifts: cross b_shift_bits, op {
    ignore_bins other_ops = binsof(op) intersect {[0:5]};
    ignore_bins popcnt_invalid = binsof(op) intersect {[8:15]};
  }

  // More complicated coverage could be written, but we'll leave
  // that for part 3.

endgroup : cg

```

### 大概解释一下 不一定对：
```systemverilog
  coverpoint a;

  // Same for b.
  coverpoint b;

  // Checks that your testbench exercises all valid operations.
  coverpoint op {
    bins range[] = {[0:8]};
  }

```
我们需要一个covergrop去探测A, B, 和OP。 这个covergroup的覆盖率会被打进报告里，用sample采样。验证机制目前未知？
对于OP，我们的范围是0~8。我们只在乎这几个值
cross的意思是：我们需要交叉验证对于b_shift_bits的每一个op的情况。
我么你想安定了，说，对于op而言，有两种情况下的op是没用的。分别是0~5  和 8~15的情况。


