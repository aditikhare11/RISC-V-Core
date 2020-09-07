# Building a RISC-V Core using TL-Verilog

This project contains all the information needed to build a RISC-V Core which implements the RV32I Base Instruction Set. The core is built using TL-Verilog and MakerChip. 

## Table Of Contents

- Introduction to RISC-V ISA
- Setting up the environment
- ABI
- Makerchip and TL-Verilog
  - Introduction
  -

# Introduction to RISC-V ISA

RISC-V is an open standard instruction set architecture based on established reduced instruction set computer principles. It is provided under open-source license which gives it a huge advantage when compared to other commercial ISAs. It is a simple, stable, small standard base ISA with multiple standard extensions. It was developed in UC Berkeley. 

The RISC-V ISA is defined as a base integer ISA, which must be present in any implementation, plus optional extensions to the base ISA. The base RISC-V ISA has a little-endian memory system. The standard is maintained by the RISC-V foundation. You can learn more about RISC-V [here](https://riscv.org/).

The base integer instructions as represented as **RV32I/RV64I** and they operate on integer numbers. Other extensions are as follows:
**RV64M** - multiply extension
**RV64F** and **RV64D** - single and double precision floating point extension

A core with all the above extensions will be represented as **RV64IMFD**. 

# Setting up the environment

To understand RISC-V ISA you will need a GNU GCC cross-compiler for RISC-V and a simulator. I have set this up on Ubuntu 18.04. 
There are a number of cross-compilation tools available but I installed mine from the following sources:
1. [RISC-V GNU Toolchain](http://hdlexpress.com/RisKy1/How2/toolchain/toolchain.html)
2. [RISC-V ISA Simulator - Spike](https://github.com/kunalg123/riscv_workshop_collaterals)

Testing with a simple code:
A basic C code called [sum1ton.c](https://github.com/aditikhare11/RISC-V-Core/blob/master/Codes/sum1ton.c) which adds numbers from 1 to n is used. 

**1. To compile with RISC-V GCC compiler:**
```
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
```
**2. To see disassembled file:**
```
riscv64-unknown-elf-objdump -d sum1ton.o | less
```
![Steps 1 and 2](https://github.com/aditikhare11/RISC-V-Core/blob/master/Outputs/sum1ton-compile.png)

![Output of 2](https://github.com/aditikhare11/RISC-V-Core/blob/master/Outputs/sum1ton-disassemble.png)

Number of instructions are 15 in this case. This number can be decreased by using -Ofast instead of -O1 during compiling. To see reduced instructions use
```
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
riscv64-unknown-elf-objdump -d sum1ton.o | less
```
In this case, the number of instructions are 12. 

**3. To compile:**
```
spike pk sum1ton.o
```
![Spike 1](https://github.com/aditikhare11/RISC-V-Core/blob/master/Outputs/sum1ton-spike1.png)

**4. To debug using spike:**
```
spike -d pk sum1ton.o
```
After running the above code line a number of things can be done as demonstrated in the image below. The code can be manually debugged, part of it can be run and contents of registers can be checked.

```:q``` to quit. 

![Spike 2](https://github.com/aditikhare11/RISC-V-Core/blob/master/Outputs/sum1ton-spike2.png)

# ABI 

Application Binary Interface aka system call interface is used by the application program to access registers. RISC-V ABI defines standard functions for registers which allows for software interoperability. 

In RISC-V specification there are 32 registers from x(0) to x(31) whose width is defined by XLEN which can be 32/64  for RV32/RV64 respectively. 

The data can be loaded from memory to registers or directly sent. Memory is byte addressable. RISC-V belongs to the little endian memory addressing system. 

Here is the RISC-V calling convention. [Image source:riscv](https://riscv.org/).

![ABI](https://github.com/aditikhare11/RISC-V-Core/blob/master/RISC-V/ABI.PNG)

Testing ABI call using [1to9_custom.c](https://github.com/aditikhare11/RISC-V-Core/blob/master/Codes/1to9_custom.c) and [load.S](https://github.com/aditikhare11/RISC-V-Core/blob/master/Codes/load.S) to find sum of numbers from 1 to 9. 

![ABI-call](https://github.com/aditikhare11/RISC-V-Core/blob/master/Outputs/abicall.png)



