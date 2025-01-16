```systemverilog
  all_opcodes : coverpoint instr.i_type.opcode;
  all_funct7 : coverpoint funct7_t'(instr.r_type.funct7);
  all_funct3 : coverpoint instr.r_type.funct3;
  all_funct3_2 : coverpoint load_funct3_t'(instr.i_type.funct3);
  all_funct3_3 : coverpoint store_funct3_t'(instr.s_type.funct3);
  all_funct3_4 : coverpoint arith_funct3_t'(instr.r_type.funct3);
  all_regs_rs1 : coverpoint reg_rs1'(instr.r_type.rs1);
  all_regs_rs2 : coverpoint reg_rs2'(instr.r_type.rs2);
  
  funct3_cross : cross instr.i_type.opcode, instr.i_type.funct3 {

    // We want to ignore the cases where funct3 isn't relevant.

    // For example, for JAL, funct3 doesn't exist. Put it in an ignore_bins.
    ignore_bins JAL_FUNCT3 = funct3_cross with (instr.i_type.opcode == op_jal);

    // TODO:  What other opcodes does funct3 not exist for? Put those in
    // ignore_bins.
    ignore_bins LUI_FUNCT3 = funct3_cross with (instr.i_type.opcode == op_lui);
    ignore_bins AUIPC_FUNCT3 = funct3_cross with (instr.i_type.opcode == op_auipc);
    //ignore_bins JALR_FUNCT3 = funct3_cross with (instr.i_type.opcode == op_jalr);

    // Branch instructions use funct3, but only 6 of the 8 possible values
    // are valid. Ignore the other two -- don't include them in the coverage
    // report. In fact, if they're generated, that's an illegal instruction.
    ignore_bins BR_FUNCT3 = funct3_cross with
    (instr.i_type.opcode == op_br
     && !(instr.i_type.funct3 inside {beq, bne, blt, bge, bltu, bgeu}));


    // TODO: You'll also have to ignore some funct3 cases in JALR, LOAD, and
    // STORE. Write the illegal_bins/ignore_bins for those cases.
    //  illegal_bins JALR_FUNCT3 = funct3_cross with 
    // (instr.i_type.opcode == op_jalr 
    // && !(instr.i_type.funct3 inside{ op_funct3 }));
    illegal_bins JALR_FUNCT3 = funct3_cross with
    (instr.i_type.opcode == op_jalr
    && !(instr.i_type.funct3 == 3'b000));


    illegal_bins LOAD_FUNCT3 = funct3_cross with 
    (instr.i_type.opcode == op_load
    && !(instr.i_type.funct3 inside{lb, lh, lw, lbu, lhu})); 

    illegal_bins STORE_FUNCT3 = funct3_cross with 
    (instr.i_type.opcode == op_store
    && !(instr.i_type.funct3 inside{sb, sh, sw})); 
  }

  // Coverpoint to make separate bins for funct7.
  coverpoint instr.r_type.funct7 {
    bins range[] = {[0:$]};
    ignore_bins not_in_spec = {[1:31], [33:127]};
  }

  // Cross coverage for funct7.
  funct7_cross : cross instr.r_type.opcode, instr.r_type.funct3, instr.r_type.funct7 {

    // No opcodes except op_reg and op_imm use funct7, so ignore the rest.
    ignore_bins OTHER_INSTS = funct7_cross with
    (!(instr.r_type.opcode inside {op_reg, op_imm}));

    // TODO: Get rid of all the other cases where funct7 isn't necessary, or cannot
    // take on certain values.
    ignore_bins FUN7_VALUE = funct7_cross with
    (!(instr.r_type.funct7 inside {base, variant}));
    ignore_bins IMMI_INSTS = funct7_cross with
    (((instr.r_type.funct3 inside {add, slt, sltu, axor, aor, aand}) 
    && (instr.r_type.opcode == op_imm)));
    ignore_bins IMM_SLLI = funct7_cross with
    ((instr.r_type.funct3 inside {sll}) && !(instr.r_type.funct7 inside {base})); 
    ignore_bins REG_BASE_INST = funct7_cross with
    ((instr.r_type.funct3 inside{sll , slt, sltu, axor, aor, aand}) 
    && (instr.r_type.funct7 inside { variant})
    && instr.r_type.opcode == op_reg);
  }
endgroup : instr_cg
```
验证思路：首先，coverpoint是i_type的opcode, r_type的func7要被强转成base 和varient 两种格式。coverpoints的r_type funct3t, store_funct3t reg_rs1, reg_rs2 强转+潜在过滤。      
funct3_cross instr.itype_opcode 和 instr.i_type.funct3 需要ignore一些funct3，例如opjal， lui auipc，需要忽略掉funct3， op_br 需要限制部分的funct3        
illegal bin也还要加上一些别的opcode 的funct3     
funct3_cross 也要滤掉一些，例如 inst_rtype 的opcode， funct3 和funct7， 首先需要滤掉除了op-imm 和op-reg 首先去掉base 和variant，然后一个个排除。 按照variant和base。    



