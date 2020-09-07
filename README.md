# Building a RISC-V Core using TL-Verilog

This project contains all the information needed to build a RISC-V Core which implements the entire RV32I Base Instruction Set (except FENCE, ECALL, EBREAK). The core is built using TL-Verilog. 

## Table Of Contents

- Introduction to RISC-V ISA
- Setting up the environment


# Introduction to RISC-V ISA

RISC-V is an open standard instruction set architecture based on established reduced instruction set computer principles. It is provided under open-source license which gives it a huge advantage when compared to other commercial ISAs. It is a simple, stable, small standard base ISA with multiple standard extensions. It was developed in UC Berkeley. 

The RISC-V ISA is defined as a base integer ISA, which must be present in any implementation, plus optional extensions to the base ISA. The base RISC-V ISA has a little-endian memory system. The standard is maintained by the RISC-V foundation. You can learn more about RISC-V [here](https://riscv.org/).

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
**4. To debug using spike:**
```
spike -d pk sum1ton.o
```
After running the above code line a number of things can be done as demonstrated in the image below. The code can be manually debugged, part of it can be run and contents of registers can be checked.

```:q``` to quit. 





![Alt Text](url)
