# Lab1 Report

B07901069 電機四 劉奇聖

## Modules Explanation

### `Adder.v`

(Same as homework 4)

Adder module reads two 32 bits signed integer as inputs and output the sum of these two integers. This module directly assign the sum of these two integers to output.

### `MUX2.v`

(Same as `MUX32` module in homework 4)

MUX2 module reads two 32 bits data and a select signal as input, and outputs one of these two data based on the select signal. This module directly assigns the output wire with the verilog ternary conditional operator. If the select signal is 1, output data 2. Else output data 1.

### `MUX4.v`

MUX4 module reads four 32 bits data and a 2-bit select signal as input, and outputs one of these four data based on the select signal. This module directly assigns the output wire with the verilog ternary conditional operator. If the select signal is 00, output data 1. If the select signal is 01, output data 2. If the select signale is 10, output data 3. Else output data 4.

### `Sign_Extend.v`

Sign_Extend module reads the 32 bit instruction as input, and outputs its sign-extended 32 bit immediate as output based on the instruction type. It first examines the opcode of the instruction. If the opcode indicates that this instruction is `sw`, then `data_o` will be the concatenation of 20 bits of `funct7[6]` and funct7 and `RDaddr`. If the opcode indicates that this instruction is `beq`, then `data_o` will be the concatenation of 21 bits of `funct7[6]` and `RDaddr[0]` and `funct7[5:0]` and `RDaddr[4:1]`. Otherwise `data_o` will be the concatenation of 20 bits of `funct7[6]` and `funct7` and `RS2addr`.

### `Control.v`

Control module reads 7 bit opcode and 1 bit NoOp signal as input, and outputs 1 bit RegWrite, 1 bit MemtoReg, 1 bit MemRead, 1 bit MemWrite, 2 bit ALUOp, 1 bit ALUSrc and 1 bit Branch signals. If NoOp is 1, then all output signals will be zero. This can be observed from `~NoOp_i &&` in all 1 bit output signals and `(NoOp_i) ? 2'b00` in ALUOp_o assignment. `~NoOp_i &&` statement indicates that if `NoOp_i` is 1, then the output will be the logical-and of 0 and another signal, which is always 0. In `ALUOp_o`, if `NoOp_i` is 1, then `ALUOp_o` will be `2'b00`.

If `NoOp_i` is 0, then this module assigns `RegWrite_o` to 1 if the opcode is not store or branch operation. It assigns `MemtoReg_o` to 1 if the opcode is not R-type or I-type instructions. It assigns `MemRead_o` to 1 if the opcode is `ld`. It assigns `MemWrite_o` to 1 if the opcode is `sd`. It assigns `ALUSrc_o` to 1 if the opcode is not R-type or `beq`. It assigns `Branch_o` to 1 if the opcode is `beq`.

For `ALUOp_o`, it assigns it to `ALU_Control.ALUOP_RType` if the opcode is R-type instruction. It assigns it to `ALU_Control.ALUOP_IType` if the opcode is I-type instruction. It assigns it to `ALU_Control.ALUOp_Subtract` if the opcode is `beq`. Otherwise it assigns `ALUOp_o` to `ALU_Control.ALUOp_Add`.

### `ALU_Control.v`

ALU_Control module reads 10 bit `funct_i` as input, this input is generated by concatenating `funct7` and `funct3`. This module also reads 2 bit ALUOp and outputs 3 bit ALUCtrl signal. This module first separate `funct7` and `funct3` from `funct_i`. Next assign `ALUCtrl_o` based on `ALUOp_i`, `funct7`, and `funct3`.

- If `ALUOp_i` is equal to `ALUOp_Add`, then assign `ALUCtrl_o` with `ALU.ALU_ADD`.
- If `ALUOp_i` is equal to `ALUOp_Subtract`, then assign `ALUCtrl_o` with `ALU.ALU_SUB`.
- If `ALUOp_i` is equal to `ALUOp_RType` and `funct7` is equal to `0000000` and `funct3` is equal to `111`, then assign `ALUCtrl_o` with `ALU.ALU_AND`.
- If `ALUOp_i` is equal to `ALUOp_RType` and `funct7` is equal to `0000000` and `funct3` is equal to `100`, then assign `ALUCtrl_o` with `ALU.ALU_XOR`.
- If `ALUOp_i` is equal to `ALUOp_RType` and `funct7` is equal to `0000000` and `funct3` is equal to `001`, then assign `ALUCtrl_o` with `ALU.ALU_SLL`.
- If `ALUOp_i` is equal to `ALUOp_RType` and `funct7` is equal to `0000000` and `funct3` is equal to `000`, then assign `ALUCtrl_o` with `ALU.ALU_ADD`.
- If `ALUOp_i` is equal to `ALUOp_RType` and `funct7` is equal to `0100000` and `funct3` is equal to `000`, then assign `ALUCtrl_o` with `ALU.ALU_SUB`.
- If `ALUOp_i` is equal to `ALUOp_RType` and `funct7` is equal to `0000001` and `funct3` is equal to `000`, then assign `ALUCtrl_o` with `ALU.ALU_MUL`.
- If `ALUOp_i` is equal to `ALUOp_IType` and `funct3` is equal to `000`, then assign `ALUCtrl_o` with `ALU.ALU_ADD`.
- If `ALUOp_i` is equal to `ALUOp_IType` and `funct7` is equal to `0100000` and `funct3` is equal to `101`, then assign `ALUCtrl_o` with `ALU.ALU_SRA`.

### `ALU.v`

(Same as homework 4)

ALU module reads two 32 bit signed integers and 3 bit ALUCtrl as input, and outputs one 32 bit signed integer and one 1 bit zero signal. This module assigns `data_o` to different output based on `ALUCtrl`. This module assigns `Zero_o` to 1 if `ALUCtrl_i` is equal to `ALU_SUB` and `data1_i` is equal to `data_2_i` else 0.

If `ALUCtrl_i` is equal to `ALU_AND`, then assigns `data_o` with the bitwise and of `data1_i` and `data2_i`.
If `ALUCtrl_i` is equal to `ALU_XOR`, then assigns `data_o` with the bitwise xor of `data1_i` and `data2_i`.
If `ALUCtrl_i` is equal to `ALU_SLL`, then assigns `data_o` with `data1_i` logically left shift by `data2_i` bits.
If `ALUCtrl_i` is equal to `ALU_ADD`, then assigns `data_o` with `data1_i + data2_i`.
If `ALUCtrl_i` is equal to `ALU_SUB`, then assigns `data_o` with `data1_i - data2_i`.
If `ALUCtrl_i` is equal to `ALU_MUL`, then assigns `data_o` with `data1_i * data2_i`.
If `ALUCtrl_i` is equal to `ALU_SRA`, then assigns `data_o` with `data1_i` arithematically right shift by `data2_i[4:0]` bits.

### `Pipeline_Registers.v`

IF_ID_Registers module reads 1 bit clock signal, 1 bit reset signal, 1 bit Stall signal, 1 bit Flush signal, 32 bits PC output, 32 bits instruction as input, and outputs 32 bits PC output, 32 bits instruction. This module declare all its output as register type. Whenever the clock signal or reset signal is on postive edge, it triggers the always block. The always block first checks that if the reset signal is 1, then it assigns all the output with 0. If the Flush signal is 1, then it assigns the instruction output with 0. If the Stall signal is 0, then it assigns all the output registers with the corresponding next-state input signals. So if the Stall signal is 1, the values in output registers will not change.

ID_EX_Registers module reads 1 bit clock signal, 1 bit reset signal, 1 bit RegWrite signal, 1 bit MemtoReg signal, 1 bit MemRead signal, 1 bit MemWrite signal, 2 bit ALUOp, 1 bit ALUSrc, 32 bit RS1data, 32 bit RS2data, 32 bit immediate, 10 bit funct, 5 bit RS1addr, 5 bit RS2addr, 5 bit RDaddr as input, and outputs 1 bit RegWrite, 1 bit MemtoReg, 1 bit MemRead, 1 bit MemWrite, 2 bit ALUOp, 1 bit ALUSrc, 32 bit RS1data, 32 bit RS2data, 32 immediate, 10 bit funct, 5 bit RS1addr, 5 bit RS2addr, 5 bit RDaddr. This module declare all its output as register type. Whenever the clock signal or reset signal is on postive edge, it triggers the always block. The always block first checks that if the reset signal is 1, then it assigns all the output with 0. Otherwise it assigns all the output registers with the corresponding next-state input signals.

EX_MEM_Registers module reads 1 bit clock signal, 1 bit reset signal, 1 bit RegWrite, 1 bit MemtoReg, 1 bit MemRead, 1 bit MemWrite, 32 bit ALUResult, 32 bit RS2data, 5 bit RDaddr as input, and outputs 1 bit RegWrite, 1 bit MemtoReg, 1 bit MemWrite, 32 bit ALUResult, 32 bit RS2data, 5 bit RDaddr. This module declare all its output as register type. Whenever the clock signal or reset signal is on postive edge, it triggers the always block. The always block first checks that if the reset signal is 1, then it assigns all the output with 0. Otherwise it assigns all the output registers with the corresponding next-state input signals.

MEM_WB_Registers module reads 1 bit clock signal, 1 bit reset signal, 1 bit RegWrite signal, 1 bit MemtoReg signal, 32 bit ALUResult, 32 bit MemData, 5 bit RDaddr as input, and outputs 1 bit RegWrite, 1 bit MemtoReg, 32 bit ALUResult, 32 bit MemData, 5 bit RDaddr. This module declare all its output as register type. Whenever the clock signal or reset signal is on postive edge, it triggers the always block. The always block first checks that if the reset signal is 1, then it assigns all the output with 0. Otherwise it assigns all the output registers with the corresponding next-state input signals.

### `Forwarding_Unit.v`

Forwarding_Unit module reads 5 bit EX stage RS1addr, 5 bit EX stage RS2addr, 1 bit MEM stage RegWrite signal, 5 bit MEM stage RDaddr, 1 bit WB RegWrite signal, 5 bit WB stage RDaddr as input, and outputs 2 bit Forward_A and 2 bit Forward_B signals.
This module assigns Forward_A to 10 if MEM_RegWrite is 1 and MEM_RDaddr_i is not 0 and MEM_RDaddr is equal to EX_RS1addr. It assigns Forward_A to 01 if WB_RegWrite is 1 and WB_RDaddr is not 0 and the conditions to assign Forward_A to 10 is not true and WB_RDaddr is equal to EX_RS1addr. Otherwise it assigns Forward_A to 00.
This module assigns Forward_B to 10 if MEM_RegWrite is 1 and MEM_RDaddr_i is not 0 and MEM_RDaddr is equal to EX_RS2addr. It assigns Forward_B to 01 if WB_RegWrite is 1 and WB_RDaddr is not 0 and the conditions to assign Forward_B to 10 is not true and WB_RDaddr is equal to EX_RS2addr. Otherwise it assigns Forward_B to 00.

### `Hazard_Detection.v`

Hazard_Detection module reads 5 bit ID stage RS1addr, 5 bit ID stage RS2addr, 1 bit EX stage MemRead signal, 5 bit EX stage RDaddr as inputs, and outputs 1 bit PCWrite, 1 bit Stall, 1 bit NoOp signals. This module assigns `Stall_o` to 1 if EX_MemRead is 1 and EX_RDaddr is equal to ID_RS1addr or ID_RS2addr. It assigns PCWrite with the inverse of Stall_o. It assigns NoOp_o with the same value of Stall_o.

### `CPU.v`

CPU module reads 1 bit clock signal, 1 bit reset signal, and 1 bit start signal as input. This module only declare wires and assign them to each submodules based on the data path figure.

This module assigns `Flush` to 1 if `Branch` signal is 1 and `ID_RS1data` is equal to `ID_RS2data`.

This module extract `ID_funct7`, `ID_RS2addr`, `ID_RS1addr`, `ID_funct3`, `ID_RDaddr`, `ID_opcode` from the `ID_instruction` wire. Then it assigns `ID_funct` with the concatenation of `ID_funct7` and `ID_funct3`.

Next it connect modules as following:

- Connect clk_i, rst_i, Stall, Flush, IF_pc_o, IF_instruction, ID_pc_o, ID_instruction wires to `IF_ID_Registers` module.
- Connect clk_i, rst_i, ID_RegWrite, ID_MemtoReg, ID_MemRead, ID_MemWrite, ID_ALUOp, ID_ALUSrc, ID_RS1data, ID_RS2data, ID_immediate, ID_funct, ID_RS1addr, IDRS2addr, ID_RDaddr, EXRegWrite, EX_MemtoReg, EX_MemRead, EX_ALUOp, EX_ALUSrc, EX_RS1data, EX_RS2data, EX_immediate, EX_funct, EX_RS1addr, EX_RS2addr, EX_RDaddr wires to `ID_EX_Registers` module.
- Connect clk_i, rst_i, EX_RegWrite, EX_MemtoReg, EX_MemRead, EX_MemWrite, EX_ALUResult, ALUInput_MUX_4_2_connect, EX_RDaddr, MEM_RegWrite, MEM_MemtoReg, MEM_MemRead, MEM_MemWrite, MEM_ALUResult, MEM_RS2data, MEMRDaddr wires to `EX_MEM_Registers` module.
- Connect clk_i, rst_i, MEM_RegWrite, MEM_MemtoReg, MEM_ALUResult, MEM_MemData, MEM_RDaddr, WB_RegWrite, WB_MemtoReg, WB_ALUResult, WB_MemData, WB_RDaddr wires to `MEM_WB_Registers` module.
- Connect pc_add_4, Branch_addr, Flush, pc_i wires to `MUX2` module named `MUX2_pc_i`.
- Connect clk_i, rst_i, start_i, PCWrite, pc_i, IF_pc_o to `PC` module.
- Connect IF_pc_o, IF_instruction to `Instruction_Memory` module.
- Connect IF_pc_o, constant 4, pc_add_4 to `Adder` module named `Add_PC`.
- Connect NoOp, ID_opcode, IDRegWrite, ID_MemtoReg, ID_MemRead, ID_MemWrite, ID_ALUOp, ID_ALUSrc, Branch wires to `Control` module.
- Connect clk_i, ID_RS1addr, ID_RS2addr, WB_RDaddr, RDdata, WB_RegWrite, ID_RS1data, ID_RS2data wires to `Registers` module.
- Connect ID_instruction, ID_immediate to `Sign_Extend` module.
- Connect ID_immediate left shift by 1 bit, ID_pc_o, Branch_addr to `Adder` module named `Branch_addr_adder`.
- Connect EX_RS1data, RDdata, MEM_ALUResult, ForwardA, ALUInput1 to `MUX4` module named `MUX4_ALUInput1`.
- Connect EX_RS2data, RDdata, MEM_ALUResult, ForwardB, ALUInput_MUX_4_2_connect to `MUX4` module named `MUX4_ALUInput2`.
- Connect ALUInput_MUX_4_2_connect, EX_immediate, EX_ALUSrc, ALUInput2 to `MUX2` module named `MUX2_ALUInput2`.
- Connect EX_funct, EX_ALUOp, ALUCtrl to `ALU_Control` module.
- Connect ALUInput1, ALUInput2, ALUCtrl, EX_ALUResult to `ALU` module.
- Connect clk_i, MEM_ALUResult, MEM_MemRead, MEM_MemWrite, MEM_RS2data, MEM_Memdata to `Data_Memory` module.
- Connect WB_ALUResult, WB_MemData, WB_MemtoReg, RDdata to `MUX2` module named `MUX2_RDdata`.
- Connect EX_RS1addr, EX_RS2addr, MEM_RegWrite, MEM_RDaddr, WB_RegWrite, WB_RDaddr, ForwardA, ForwardB wires to `Forwarding_Unit` module.
- Connect ID_RS1addr, ID_RS2addr, EX_MemRead, EX_RDaddr, PCWrite, Stall, NoOp wires to `Hazard_Detection` module.

## Difficulties Encountered and Solutions in This Lab

### Difficulty 1

The testcases provided by TA has no `beq` instruction.

### Solution 1

I wrote custom testcases containing `beq` instruction for debuging.

### Difficulty 2

Flush and Stall are hard to debug.

### Solution 2

Use gtkwave to debug and carefully trace each signal and observe the values of pipeline registers.

## Development Environment

My OS is "Linux Mint 20.2 Cinnamon". The compiler is `iverilog` version 10.3. I use VSCode to write this homework.