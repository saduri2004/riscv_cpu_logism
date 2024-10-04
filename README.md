# Project 3: RISC-V Processor Design

## Overview

In this project, I designed and implemented a CPU capable of executing RISC-V instructions. The project was divided into two main parts:

1. **Part A:** I implemented the CPU to execute the `addi` instruction, focusing on designing individual components such as the ALU, Register File, Immediate Generator, and Datapath.
2. **Part B:** I extended the CPU to support additional instructions, including I-type, R-type, B-type, S-type, U-type instructions, and implemented a 2-stage pipelined architecture.

## Part A: Designing the Skeleton CPU (addi Instruction)

In Part A, my goal was to implement a basic CPU that could execute the `addi` instruction. This involved building the Arithmetic Logic Unit (ALU), Register File, Immediate Generator, and the Datapath.

### Task 1: Arithmetic Logic Unit (ALU)

In `alu.circ`, I implemented the following ALU operations:

| ALUSel Value | Instruction | Description |
|--------------|-------------|-------------|
| 0            | `add`       | Result = A + B |
| 1            | `sll`       | Result = A << B |
| 2            | `slt`       | Result = (A < B) ? 1 : 0 |
| 4            | `xor`       | Result = A ^ B |
| 5            | `srl`       | Result = (unsigned) A >> B |
| 6            | `or`        | Result = A \| B |
| 7            | `and`       | Result = A & B |
| 8            | `mul`       | Result = (A * B)[31:0] |
| 9            | `mulh`      | Result = (A * B)[63:32] |
| 11           | `mulhu`     | Result = (A * B)[63:32] (unsigned) |
| 12           | `sub`       | Result = A - B |
| 13           | `sra`       | Result = (signed) A >> B |

I used Logisim components to implement these operations. The `mulh` and `mulhu` operations extract the upper 32 bits of the multiplication result, and shifts only use the lower 5 bits of B.

### Task 2: Register File

In `regfile.circ`, I implemented a 32-register file with the following operations:

- **Inputs:**
  - `ReadIndex1`: Selects the register to output on `ReadData1`.
  - `ReadIndex2`: Selects the register to output on `ReadData2`.
  - `WriteIndex`: Selects the register to write to.
  - `WriteData`: Data to be written to the selected register.
  - `RegWEn`: Enables writing to the register.
  - `clk`: Clock signal.

- **Outputs:**
  - `ReadData1`, `ReadData2`: Data from the selected registers.
  - Fixed output for registers `ra`, `sp`, `t0`, `t1`, `t2`, `s0`, `s1`, `a0`.

I ensured that the `x0` register always contains zero, even if written to.

### Task 3: Immediate Generator

In `imm-gen.circ`, I implemented the Immediate Generator to generate 32-bit sign-extended immediates for the `addi` instruction.

- **Input:**
  - `Instruction`: The 32-bit instruction word.
  - `ImmSel`: Selector signal to choose the immediate type (for now, I only handled I-type).

- **Output:**
  - `Immediate`: The sign-extended immediate value.

### Task 4: Datapath

In `cpu.circ`, I built the datapath for the `addi` instruction. This involved connecting the program counter (PC), instruction memory (IMEM), register file, ALU, and immediate generator. I completed the following tasks:

1. Fetch the instruction from memory.
2. Decode the instruction and generate control signals.
3. Perform the addition using the ALU.
4. Write the result back into the register file.

## Part B: Supporting More Instructions and Pipelining

In Part B, I extended the CPU to support additional instructions, added control logic, and implemented a 2-stage pipeline.

### Task 5: I-type Instructions

I extended the CPU to support the following I-type instructions:

| Instruction | Opcode | Funct3 | Operation |
|-------------|--------|--------|-----------|
| `andi`      | 0x13   | 0x7    | rd = rs1 & imm |
| `ori`       | 0x13   | 0x6    | rd = rs1 \| imm |
| `xori`      | 0x13   | 0x4    | rd = rs1 ^ imm |
| `slli`      | 0x13   | 0x1    | rd = rs1 << imm |
| `srli`      | 0x13   | 0x5    | rd = rs1 >> imm (Zero-extend) |
| `slti`      | 0x13   | 0x2    | rd = (rs1 < imm) ? 1 : 0 |

I modified the control logic in `control-logic.circ` to generate the correct control signals for these instructions.

### Task 6: R-type Instructions

I extended the CPU to support R-type instructions like `add`, `sub`, `and`, `or`, `xor`, and `mul`. I updated the datapath to handle register-to-register operations and modified the control logic to support these instructions.

### Task 7: B-type Instructions (Branching)

I implemented branch instructions like `beq`, `bne`, `blt`, and `bge`. To handle branching, I built a branch comparator circuit (`branch-comp.circ`) that compares the values of `rs1` and `rs2` for conditional branching.

### Task 8: S-type Instructions (Memory)

I extended the CPU to support load and store instructions (`lw`, `sw`). I also implemented partial load and store logic in `partial-load-store.circ` and updated the datapath to include memory access stages.

### Task 9: Jumps and U-type Instructions

I implemented jump (`jal`, `jalr`) and U-type (`lui`, `auipc`) instructions. I modified the immediate generator to handle different immediate formats and adjusted the control logic accordingly.

### Task 10: Pipelining

I implemented a 2-stage pipeline:

1. **Instruction Fetch (IF):** Fetch the instruction from memory.
2. **Execute (EX):** This stage combines instruction decode, execute, memory access, and write-back.

To handle control hazards, I flushed the pipeline on branches and jumps by injecting no-op instructions (`addi x0, x0, 0`).
