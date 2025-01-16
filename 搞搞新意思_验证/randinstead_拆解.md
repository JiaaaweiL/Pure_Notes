# 先来拆解randinst.svh
```systemverilog
localparam NUM_TYPES = 9;

  typedef enum bit [6:0] {
    base    = 7'b0000000,
    variant = 7'b0100000
  } funct7_t;

  typedef union packed {
    bit [31:0] word;

    struct packed {
      bit [11:0] i_imm;
      bit [4:0] rs1;
      bit [2:0] funct3;
      bit [4:0] rd;
      rv32i_opcode opcode;
    } i_type;

    struct packed {
      bit [6:0] funct7;
      bit [4:0] rs2;
      bit [4:0] rs1;
      bit [2:0] funct3;
      bit [4:0] rd;
      rv32i_opcode opcode;
    } r_type;

    struct packed {
      bit [11:5] imm_s_top;
      bit [4:0]  rs2;
      bit [4:0]  rs1;
      bit [2:0]  funct3;
      bit [4:0]  imm_s_bot;
      rv32i_opcode opcode;
    } s_type;

    struct packed {
      bit [4:0] rs1;
      bit [4:0] rs2;
      bit [2:0] funct3;
      bit [6:0] imm_b_top;
      bit [4:0] imm_b_bot; 
      rv32i_opcode opcode;
    } b_type;

    struct packed {
      bit [31:12] imm;
      bit [4:0]  rd;
      rv32i_opcode opcode;
    } j_type;

  } instr_t;
```
首先，为什么NUM_TYPES是9？ 因为有9种不同的op-code。所有指令按照这九种不同的opcode可以区分。
```systemverilog
op_lui   = 7'b0110111, // load upper immediate (U type)
op_auipc = 7'b0010111, // add upper immediate PC (U type)
op_jal   = 7'b1101111, // jump and link (J type)
op_jalr  = 7'b1100111, // jump and link register (I type)
op_br    = 7'b1100011, // branch (B type)
op_load  = 7'b0000011, // load (I type)
op_store = 7'b0100011, // store (S type)
op_imm   = 7'b0010011, // arith ops with register/immediate operands (I type)
op_reg   = 7'b0110011  // arith ops with register operands (R type)
```
九种不同的op-code可以拆解成为多种type，U-type， J-type， I-type， B-type 和R-type.   
bit [31:0] word; 其实是，它采集到的事务是32个bit，然后会被一下某一种type来解释。 这是一个基础字段，用来存储所有指令的原始 32 位数据。  
instr_t 是一个 union，当我们通过某种类型（例如 i_type）访问 instr_t 时，编译器会按照该类型的字段结构来解释 word。
例如，当 instr_t 被视为 i_type 时：word[11:0] 被解析为 i_imm。word[14:12] 被解析为 funct3。word[31:25] 将被忽略，因为它们不属于 i_type 的字段。     

```systemverilog
rand instr_t instr;
  rand bit [NUM_TYPES-1:0] instr_type;

  // Make sure we have an even distribution of instruction types.
  constraint solve_order_c { solve instr_type before instr; }

  // to get 100% coverage with 500 calls to .randomize().
  // constraint solve_order_funct3_c { ... }
  rand bit [2:0] functt3;
  constraint sovle_order_funct3_c { solve functt3 before instr; }

  // Pick one of the instruction types.
  constraint instr_type_c {
    $countones(instr_type) == 1; // Ensures one-hot.
  }

```
为了最少次数的random可以造成更可能多的hit，我们需要在     
1. 在随机化instr之前，先随机化instr type；   
2. 在随机化instr之前，先随机化function3；

上面的 instr_type将会被保证是一个独热码。这个one hot 将会决定实际产生的指令
```systemverilog
constraint instr_c {
      instr.r_type.funct3 == functt3;
      // Reg-imm instructions
      instr_type[0] -> {
        instr.i_type.opcode == op_imm;

        // Implies syntax: if funct3 is sr, then funct7 must be
        // one of two possibilities.
        instr.r_type.funct3 == sr -> {
          instr.r_type.funct7 inside {base, variant};
        }

        // This if syntax is equivalent to the implies syntax above
        // but also supports an else { ... } clause.
        if (instr.r_type.funct3 == sll) {
          instr.r_type.funct7 == base;
        }
      }

      // Reg-reg instructions
      instr_type[1] -> {
        instr.r_type.funct3 inside {add, slt, sltu, aand, aor, axor, sll, sr};
        instr.s_type.opcode == op_reg;
        // instr.r_type.funct3 inside {add, slt, sltu, aand, aor, axor, sll, sr};
        if (instr.r_type.funct3 == add || instr.r_type.funct3 == sr) {
          instr.r_type.funct7 inside {base, variant};}
        else {
          instr.r_type.funct7 == base;}
      }
      
      // Store instructions -- these are easy to constrain!
      instr_type[2] -> {
        instr.s_type.opcode == op_store;
        instr.s_type.funct3 inside {sb, sh, sw};
      }

      // // Load instructions
      // instr_type[3] -> {
      //   instr.i_type.opcode == op_load;
      // TODO: Constrain funct3 as well.
      // }
      instr_type[3] -> {
        instr.i_type.opcode == op_load;
        instr.i_type.funct3 inside {lb, lh, lw, lbu, lhu};
      }
      instr_type[4] -> {
        instr.b_type.opcode == op_br;
        instr.b_type.funct3 inside {beq, bne, blt, bge, bltu, bgeu};
      }
      instr_type[5] -> {
        instr.i_type.opcode == op_jalr;
        instr.i_type.funct3 == 3'b000;//check!!!
      }
      instr_type[6] -> {
        instr.j_type.opcode == op_jal;
        // instr.j_type.funct3 == 3'b000;
      }
      instr_type[7] -> {
        instr.i_type.opcode == op_auipc;
      }
      instr_type[8] -> {
        instr.i_type.opcode == op_lui;
      }
      // TODO: Do all 9 types!
  }
```
我们规定的第一件事是： 如果instr是一个rtype的，他将会用“随机化instr之前先随机的functt3”
接着，对于独热码的每一种可能，先给分配上一个值。
例如，如果是一个imm_type的指令，需要做的事情是，：首先确保，如果funct3是sr的话，生成的指令funct7只会出现在base 和variant之间。 问就职指令集规定。   
同时，如果是r_type的funct3是sll的话，必须保证它的funct7是base；     

接着规定，如果随机的opcode是op-reg的话，则有一下一系列限定。      
funct3必须是{add, slt, sltu, aand, aor, axor, sll, sr}   并且，在add 和 hr，funct7有特殊的值，确保是加/减， 左移/右移      
同样的 ，如果是 op-store，还是store， 还是br，还是jalr， jal， auipc，lui，要做的第一件事是首先限定它的function3。    

**我们用union pack定义了我们如何解释32bit word的具体方式。这个32bit的word将会被具体的用五种不同的type进行解释。在我们随机的时候，因为我们随机了instr_type，我们就间接的决定了：我们将要随机的instr将会是哪一种op-imm。因此，实际上，对于instr的随机，会局限在instr-type确定好的op code里面，随机的东西是指令的剩余的值。**  



