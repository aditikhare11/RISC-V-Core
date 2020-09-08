# Building a RISC-V Core using TL-Verilog

This project contains all the information needed to build a RISC-V Core which implements the RV32I Base Instruction Set. The core is built using TL-Verilog and Makerchip. 

## Table Of Contents

- [Introduction to RISC-V ISA](https://github.com/aditikhare11/RISC-V-Core#introduction-to-risc-v-isa)
- [Setting up the environment](https://github.com/aditikhare11/RISC-V-Core#setting-up-the-environment)
- [ABI](https://github.com/aditikhare11/RISC-V-Core#abi)
- [Makerchip and TL-Verilog](https://github.com/aditikhare11/RISC-V-Core#makerchip-and-tl-verilog)
  - [Introduction](https://github.com/aditikhare11/RISC-V-Core#1-introduction)
  - [Implementation of a RISC-V core](https://github.com/aditikhare11/RISC-V-Core#implementation-of-a-risc-v-core)
  - [Testing the core with a testbench](https://github.com/aditikhare11/RISC-V-Core#testing-the-core-with-a-testbench)
  - [Pipelining the RISC-V core](https://github.com/aditikhare11/RISC-V-Core#testing-the-core-with-a-testbench)
- [Future work](https://github.com/aditikhare11/RISC-V-Core#future-work)
- [Acknowledgements](https://github.com/aditikhare11/RISC-V-Core#acknowledgements)

# Introduction to RISC-V ISA

RISC-V is an open standard instruction set architecture based on established reduced instruction set computer principles. It is provided under open-source license which gives it a huge advantage when compared to other commercial ISAs. It is a simple, stable, small standard base ISA with multiple standard extensions. It was developed in UC Berkeley. 

The RISC-V ISA is defined as a base integer ISA, which must be present in any implementation, plus optional extensions to the base ISA. The base RISC-V ISA has a little-endian memory system. The standard is maintained by the RISC-V foundation. You can learn more about RISC-V [here](https://riscv.org/).

The base integer instructions as represented as **RV32I/RV64I** and they operate on integer numbers. Other extensions are as follows:

- **RV64M** - multiply extension
- **RV64F** and **RV64D** - single and double precision floating point extension

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

**Application Binary Interface** aka system call interface is used by the application program to access registers. RISC-V ABI defines standard functions for registers which allows for software interoperability. 

In RISC-V specification there are 32 registers from x(0) to x(31) whose width is defined by XLEN which can be 32/64  for RV32/RV64 respectively. 

The data can be loaded from memory to registers or directly sent. Memory is byte addressable. RISC-V belongs to the little endian memory addressing system. 

Here is the RISC-V calling convention. [Image source:riscv](https://riscv.org/).

![ABI](https://github.com/aditikhare11/RISC-V-Core/blob/master/RISC-V/ABI.PNG)

Testing ABI call using [1to9_custom.c](https://github.com/aditikhare11/RISC-V-Core/blob/master/Codes/1to9_custom.c) and [load.S](https://github.com/aditikhare11/RISC-V-Core/blob/master/Codes/load.S) to find sum of numbers from 1 to 9. 

Commands:

```
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o 1to9_custom.o 1to9_custom.c load.S
spike pk 1to9_custom.o
```

![ABI-call](https://github.com/aditikhare11/RISC-V-Core/blob/master/Outputs/abicall.png)

# Makerchip and TL-Verilog

## A. Introduction

[Makerchip](http://makerchip.com/) is a free online environment by Redwood EDA for developing high-quality integrated circuits. The online platform can be used to code, compile, simulate and debug Verilog designs from a browser.  

TL-Verilog was used as the HDL of choice for this project. Projects on Makerchip can be completely designed using TL-Verilog. Transaction Level - Verilog standard is an extension of Verilog which has various advantages like simpler syntax, shorter codes and easy pipelining. You can learn more about TL-Verilog [here](http://tl-x.org/).

Timing abstract can be done in TL-Verilog. This model is specified for pipelines where the sequential elements are generated by tools from the pipelined specification. This allows for easy retiming without the risk of introduction of any functional bugs. More information on timing abstract in TL-Verilog can be found in the IEEE paper ["Timing-Abstract Circuit Design in Transaction-Level Verilog" by Steven Hoover](https://ieeexplore.ieee.org/document/8119264).


## B. Implementation of a RISC-V core

This section will walk through the different implementation steps to achieve the RISC-V core. 

**_Please note: Click on the step to be redirected to the code associated with that step. The code can be directly pasted on Makerchip.com to view the design._**

**[1. Program Counter and Program Counter adder](https://github.com/aditikhare11/RISC-V-Core/blob/master/Makerchip%20Codes/Next%20PC)**

Program Counter is a register that contains the address of the instruction being executed. Since memory is byte addressable and the instruction length is 32 bits, the Program Counter adder adds 4 bytes to the address to point to the next address. A reset input is also present which will reset the PC value to 0. 

![PC](https://github.com/aditikhare11/RISC-V-Core/blob/master/RISC-V/PC.PNG)

**[2. Instruction Fetch](https://github.com/aditikhare11/RISC-V-Core/blob/master/Makerchip%20Codes/Instruction%20Fetch)**

Here the instruction memory is added to the program. The instruction memory contains a test program which computes the sum of numbers from 1 to 9. The instruction memory read address pointer is computed from the program counter and it outputs a 32 bit instruction. (instr\[31:0])

![Fetch](https://github.com/aditikhare11/RISC-V-Core/blob/master/RISC-V/Fetch.PNG)


**[3. Instruction Decode](https://github.com/aditikhare11/RISC-V-Core/blob/master/Makerchip%20Codes/Instruction%20Decode)**

Instruction type is decoded first using 5 bits of the instruction instr\[6:2]. 

![Instruction Type](https://github.com/aditikhare11/RISC-V-Core/blob/master/RISC-V/Instructiontypedecode.PNG)

Next step is to calculate the 32 bit immediate value (imm\[31:0]) based on the instruction type. 

![Instruction Imm](https://github.com/aditikhare11/RISC-V-Core/blob/master/RISC-V/InstructionImmDecode.PNG)

Other function fields like funct7, rs2, rs1, funct3, rd and opcode are extracted based on the instruction type. At this point valid condtions need to be defined for fields like rs1, rs2, funct3 and funct7 because they are unique to only certain instruction types. 

Only 8 operations are implemented at this stage namely BEQ, BNE, BLT, BGE, BLTU, BGEU, ADDI and ADD. The other operations from the RV32I Base Instruction Set will be implemented in the later steps. To see the complete list with the associated instruction fields click [here](https://github.com/aditikhare11/RISC-V-Core#pipelining-the-risc-v-core).

**[4. Register File Read](https://github.com/aditikhare11/RISC-V-Core/blob/master/Makerchip%20Codes/Register%20File%20Read)**

The register file macro is added which can be viewed under the "NAV-TLV" tab on Makerchip. The two source register fields defined as rs1 and rs2 are fed as inputs to the register file and the outputs are the contents of the source registers. The respective enable bits are set based on the valid conditions for rs1 and rs2 as defined in the previous step. 

![File Read](https://github.com/aditikhare11/RISC-V-Core/blob/master/RISC-V/Register%20File.PNG)

**[5. ALU](https://github.com/aditikhare11/RISC-V-Core/blob/master/Makerchip%20Codes/ALU%20for%20addition)**

The Arithmetic Logic Unit is the component that computes the result based on the selected operation. At this point the code only supports ADD and ADDI operations to execute the test code. All operations will be added at a later step. 

**[6. Register File Write](https://github.com/aditikhare11/RISC-V-Core/blob/master/Makerchip%20Codes/Register%20File%20Write)**

This step is essential to provide support for instructions that have a destination register (rd) where the output must be stored. The register file enable depends on the validity of the destination register rd and the write index takes the value stored in rd. An additional condition to ignore write if the destinaton register is x0 is added. x0 register is always equal to zero in RISC-V implementation and hence must not be written to. 

**[7. Branches](https://github.com/aditikhare11/RISC-V-Core/blob/master/Makerchip%20Codes/Core%20with%20testbench)**

The final step is to add support for branch instructions. A branch target pc has to be computed and based on the branch taken value, the pc will choose the new branch target pc when required. 

![Core](https://github.com/aditikhare11/RISC-V-Core/blob/master/RISC-V/Non-pipelined%20core.PNG)

## C. Testing the core with a testbench

Now that our implementation is complete, a simple testbench statement can be added to ensure the core is working correctly. 
When the following line of code is added on Makerchip, the simulation will pass only if the value stored in r10 = sum of numbers from 1 to 9. 

```
*passed = |cpu/xreg[10]>>5$value == (1+2+3+4+5+6+7+8+9);
```
In the instruction memory, r10 has been used to store the sum. The simulation passed message can be seen under the "Log" tab and the asm file to compute sum can be viewed in the start under the "Editor" tab
[Click here](https://github.com/aditikhare11/RISC-V-Core/blob/master/Makerchip%20Codes/Core%20with%20testbench) to view to design.  

## D. Pipelining the RISC-V core

The RISC-V core designed is divided into 5 pipeline stages. Pipelining in Makerchip is extremely simple. To define a pipeline use the following syntax:
```
|<pipeline_name>
  @<pipeline_stage>
    instruction1 in the current stage
    instruction2 in the current stage
    .
    .
  @<pipeline_stage>
    instruction1 in the current stage
    instruction2 in the current stage
    .
    .
```
Staging in a pipeline is a physical attribute with no impact to behaviour. 
At this point support for register file bypass is provided. All the instructions present in the RV32I base instruction set are implemented apart from FENCE, ECALL and EBREAK. All the instructions implemented are shown in the table below which was obtained from [riscv.org](). 

![ISA](https://github.com/aditikhare11/RISC-V-Core/blob/master/RISC-V/rv32instructionset.PNG)

Load/store and jump support is added along with the following two extra lines of code to test load and store.
```
m4_asm(SW, r0, r10, 10000)
m4_asm(LW, r17, r0, 10000)
```
The snapshot of the final core can be seen below. [Click here](http://makerchip.com/sandbox/0wpfLhK8v/0vgh7NL) to view the final design on Makerchip. Additionally you can also paste the code on this [link](https://github.com/aditikhare11/RISC-V-Core/blob/master/Makerchip%20Codes/Final%20Pipelined%20RISC-V%20Core) directly on [makerchip.com](http://makerchip.com/) to view the project.

![Final Core](https://github.com/aditikhare11/RISC-V-Core/blob/master/RISC-V/FinalCore.PNG)

# Future Work 

This project was done as a part of the [**RISC-V based MYTH (Microprocessor for You in Thirty Hours)**](https://www.vlsisystemdesign.com/riscv-based-myth/) workshop conducted by **Kunal Ghosh** and **Steve Hoover**. The current project implements almost the entire RV32I base instruction set. Future work involves modifying the current design to implement support for the remaining operations and also implementation of other standard extensions like M, F and D. 

# Acknowledgements

- [Kunal Ghosh](https://github.com/kunalg123), Co-founder, VSD Corp. Pvt. Ltd.
- [Steve Hoover](https://github.com/stevehoover), Founder, Redwood EDA

