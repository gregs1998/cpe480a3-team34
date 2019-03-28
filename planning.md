# Planning Doc for CPE 480 Assignment 3 - Team 34
This *markdown* file can be used to detail plans for the assignment. It will serve as a good reference when preparing the implementor's notes, as well. Code written here is pseudo-verilog, unless otherwise noted

## ALU Definition

```pesudo-verilog
module alu(result, op, in1, in2); //TODO: How to implement floating point stuff from Dietz?
  output reg `WORD result; // `WORD = [15:0], I assume
  input wire `OP op; // `OP = opcode size
  input wire `WORD in1, in2; // in1, in2 are input operands for ALU module
  FPmul(resmul,...); // Instantiate FP mul from Dietz, **resmul** is the output of the mul FP module
  FPadd(resadd,...);
  FPrecip(resrecip,...);
  FPshift(resshift,...);
  FPi2f(resi2f,...);
  FPf2i(resf2i,...);
  always@(op,in1,in2) begin
    case(op):
      `OPadd: result = in1 + in2; end
      `OPand: result = in1 & in2; end
      `OPany: result = |in1; end
      `OPor: result = in1 | in2; end
      `OPshr: result = in1 >> 1; end
      `OPxor: result = in1 ^ in2; end
      `OPFPmul: result = resmul; end
      `OPFPadd: result = resadd; end
      `OPFPdiv: result = resrecip; end
      `OPFPshift: result = resshift; end
      `OPFPi2f: result = resi2f; end
      `OPFPf2i: result = resf2i; end
    end
endmodule // Note: This is pseudo-code and is incomplete.
```

## Pre-Pipeline
This will be executed outside the pipeline stages

```pseudo-verilog
ir = mainmem[pc]
alu alu00(res0, s1op0, s1val0, regfile[0]); // This ALU is used in stage 2 for 0th position instruction
alu alu10(res1, s1op1, s1val1, regfile[1]); // This ALU is used in stage 2 for 1st position instruction
alu alu01(resfinal0, s2op0, s2val0, regfile[0]); // This ALU is used in stage 3 for 0th position instruction
alu alu11(resfinal1, s2op1, s2val1, regfile[1]); // This ALU is used in stage 3 for 1st position instruction
```

## Pipeline

### Stage 0

```pseudo-verilog
s0source0 = ir[10:8]; // Stage 0 output, 0th instruction operand register number
s0source1 = ir[2:0]; // Stage 0 output, 1st instruction operand register number
s0op0 = ir[15:11]; // Stage 0 output, 0th instruction opcode
s0op1 = ir[7:3]; // Stage 0 output, 1st instruction opcode
pc = newpc; // Update PC from previous stage 4
```

### Stage 1
#### How does dependency handling work?

There are four potential dependencies:
```
0 -> 0
0 -> 1
1 -> 0
1 -> 1
```

```pseudo-verilog
s1val0 = regfile[s0source0]; // Stage 1 output, read 0th instruction source register. s1val0 contains a value, not register number
s1val1 = regfile[s0source1]; // Stage 1 output, read 1st instruction source register. s1val1 contains a value, not register number
if(s1source0 == s0source0 || s1source0 == s0source1) begin // checks 0th instruction slot for dependencies
  s1source0 = NOP; ??
  s1op0 = NOP;
  end // Whenever a dependency (data hazard) occurs in 0th instruction slot, insert NOP as output of Stage 1
 else begin
  s1source0 = s0source0;
  s1op0=s0op0;
  end // Whenever no dependency, 0th instruction output opcode gets assigned opcode from stage 0 (inst. continues thru pipeline)

if(s1source1 == s0source1 || s1source1 == s0source1) begin // checks 1st instruction slot for dependencies
  s1source1 = NOP; ??
  s1op1 = NOP;
  end // Whenever a dependency (data hazard) occurs in 1st instruction slot, insert NOP as output of Stage 1
 else begin
  s1source1 = s0source1;
  s1op1=s0op1;
  end // Whenever no dependency, 1st instruction output opcode gets assigned opcode from stage 0 (inst. continues thru pipeline)
