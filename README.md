# Introduction  
RIDECORE (RIsc-v Dynamic Execution CORE) is an Out-of-Order processor written in Verilog HDL. RIDECORE implements RISC-V, an open-source ISA that was originally designed by researchers at UC Berkeley (<http://www.riscv.org>).

RIDECORE's microarchitecture is based on "Modern Processor Design: Fundamentals of Superscalar Processors" (<https://www.waveland.com/browse.php?t=624&r=d|259>). Thus, we recommend users to read this book and our document (doc/ridecore_document.pdf) before using RIDECORE.

Our FPGA prototyping efforts have so far resulted in a prototype for Xilinx VC707 board. This prototype can operate at a clock frequency of 50MHz (Vivado 2015.2 is used for synthesizing and implementing the design).

Some of the hardware modules in RIDECORE are based on vscale (<https://github.com/ucb-bar/vscale>).

# RV32I Conformance Status

RIDECORE is **not a conforming implementation of the RV32I base integer ISA**.
This comparison uses ratified RV32I version 2.1 from the RISC-V unprivileged
instruction-set specification. RIDECORE implements a reduced RV32I-style
instruction set together with part of the M integer multiplication extension;
the repository does not declare a supported RISC-V ISA version.

The current implementation differs from RV32I in the following ways:

* **Loads and stores are word-only.** `LW` and `SW` are implemented, but the
  RV32I `LB`, `LH`, `LBU`, and `LHU` loads and `SB` and `SH` stores are not.
  The decoder may accept these encodings, but the execution and memory paths do
  not perform the required width conversion or sign extension.
* **`FENCE` is missing.** The RV32I `MISC-MEM` instruction is not implemented.
* **`ECALL` and `EBREAK` are missing.** RV32I `SYSTEM` instructions are not
  implemented.
* **There is no architectural trap or exception mechanism.** Illegal and
  reserved encodings do not raise an illegal-instruction exception. The
  decoder produces an `illegal_instruction` signal, but it is not consumed by
  the pipeline.
* **Instruction-address alignment exceptions are missing.** `JAL`, `JALR`, and
  taken conditional branches do not raise the required
  instruction-address-misaligned exception when a target is not four-byte
  aligned. The `JALR` target and link arithmetic itself follows RV32I:
  `(rs1 + sext(imm)) & ~1`, and `rd` receives `pc + 4`.
* **Misaligned data accesses are not trapped or fully supported.** Software
  must use naturally aligned 32-bit accesses.
* **The M extension is incomplete.** `MUL`, `MULH`, `MULHSU`, and `MULHU` are
  implemented, but `DIV`, `DIVU`, `REM`, and `REMU` are not.

The CSR, counter, and `FENCE.I` facilities are also absent. In ratified RV32I
2.1 these are separate `Zicsr`, `Zicntr`, and `Zifencei` extensions rather than
part of the base-I instruction set, but their absence means RIDECORE does not
provide a general-purpose RISC-V execution environment.

Consequently, programs must be compiled and written for this specific subset.
They must not depend on subword memory operations, trap handling, misaligned
accesses, machine-mode CSRs, or the division portion of the M extension.

# Toolchain Installation (Linux)  

* riscv-tools  
riscv-tools include a C/C++ cross compiler for RISC-V and an ISA simulator. Please refer to the following URL for the installation guide: <http://riscv.org/software-tools/>

* iverilog  
iverilog is an open-source verilog simulator. To install iverilog in Ubuntu, run the following command:  
\# apt-get install iverilog  

* memgen  
memgen is an application generating binary code for RIDECORE from ELF files. The source code of memgen can be found in toolchain/memgen-v0.9. Run the make command to compile it.

# Structure of this repository  
The structure of this repository is described in doc/struct_repository.pdf.

# How to use  

* Compiling applications and generating binary code  
Some sample applications for RIDECORE can be found in src/test/ridecore/app. For each application, you can generate the binary code (init.bin) from the C code and the startup code (in Assembly) by using the make command. To use this binary code in verilog simulations, run "make copy" to copy it to src/test/ridecore/bin.

* Simulating RIDECORE  
The simulation code of RIDECORE can be found in src/test/ridecore/sim.
You can generate an a.out executable file by running the make command.
Currently, there are three options:
  - "make last" (testbench_last.v is compiled): generates an a.out executable file for executing applications without outputting any information about the states of RIDECORE during the execution.
  - "make dbg" (testbench.v is compiled): generates an a.out executable file which reports the states of RIDECORE in every clock cycle while executing applications.
  - "make pred" (testbench_pred.v is compiled): generates an a.out executable file which reports the branch prediction information while executing applications.

