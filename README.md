# Genesis-1 Open Source CPU

Genesis-1 is an **experimental open-source CPU architecture** designed for education, research, and hardware experimentation.
The goal of this project is to build a **complete computing stack** — from CPU microarchitecture to operating system — fully open and understandable.

Genesis-1 is designed to run on **FPGA hardware** and eventually evolve into a **custom silicon implementation**.

---

## Architecture Overview

Genesis-1 is a **simple modular CPU architecture** designed to make processor design easy to understand and modify.

**Design goals**

* Simple and readable RTL
* Fully open hardware
* FPGA-friendly architecture
* Educational CPU design
* Extensible instruction set
* Build a full system (CPU → OS → Applications)

---

## System Architecture Diagram

<img width="1024" height="1024" alt="AB49D51C-0537-4417-A53E-975891A9CC5A" src="https://github.com/user-attachments/assets/7491c120-4448-43d1-94ab-a567ce7880e6" />

Architecture diagram:

![Genesis Architecture](https://viewer.diagrams.net/?tags=%7B%7D\&lightbox=1\&highlight=0000ff\&edit=_blank\&layers=1\&nav=1\&title=Genesis.drawio\&dark=auto#Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fbrilliantnarlo%2Fsystem-design%2Fmain%2FGenesis.drawio)

Or open directly:

**[https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Genesis.drawio&dark=auto#Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fbrilliantnarlo%2Fsystem-design%2Fmain%2FGenesis.drawio](https://viewer.diagrams.net/?tags=%7B%7D&lightbox=1&highlight=0000ff&edit=_blank&layers=1&nav=1&title=Genesis.drawio&dark=auto#Uhttps%3A%2F%2Fraw.githubusercontent.com%2Fbrilliantnarlo%2Fsystem-design%2Fmain%2FGenesis.drawio)**

---

## Project Goals

Genesis-1 aims to demonstrate how a full computing platform can be built from scratch:

```
Genesis CPU
   ↓
Instruction Set Architecture (ISA)
   ↓
Assembler
   ↓
Operating System
   ↓
User Applications
```

Future goals include:

* custom instruction set
* tiny operating system
* shell environment
* hardware graphics output (HDMI)
* keyboard input
* external memory
* custom PCB board

---

## Features (Planned)

* Custom instruction set
* Program counter and instruction fetch
* Arithmetic logic unit (ALU)
* Register file
* Memory interface
* Pipeline support (future)
* Hardware interrupts
* FPGA implementation
* HDMI terminal output

---

## Project Structure

```
genesis-1-open-source
│
├── rtl/                # Verilog CPU source
├── sim/                # Simulation testbenches
├── isa/                # Instruction set documentation
├── assembler/          # Genesis assembler
├── os/                 # Genesis OS
├── docs/               # Architecture documentation
└── diagrams/           # System architecture diagrams
```

---

## FPGA Targets

Genesis-1 is designed to run on common FPGA development boards.

Test targets:

* Gowin FPGA boards
* Tang Nano series
* Tang Primer
* Other Verilog-compatible boards

---

## Development Tools

Recommended toolchain:

```
Verilog Simulator
  ModelSim / QuestaSim

FPGA Toolchains
  Gowin EDA
  Yosys
  NextPNR

Diagram Tools
  draw.io / diagrams.net
```

---

## Example Development Flow

Example simulation flow:

```bash
vlib work
vlog genesis_cpu.v
vlog genesis_tb.v
vsim genesis_tb
run -all
```

---

## Future Roadmap

### Phase 1

* Basic CPU core
* ALU
* Program counter
* Instruction decode
* Memory interface

### Phase 2

* Instruction assembler
* Memory loader
* Debug interface

### Phase 3

* Video output (HDMI)
* Keyboard input
* Simple shell

### Phase 4

* Genesis Operating System
* Filesystem
* Applications

### Phase 5

* Custom PCB computer
* DDR RAM
* USB
* HDMI graphics console

---

## Long-Term Vision

The long-term vision of Genesis-1 is to build a **complete open computing platform**, similar to early home computers but fully open:

```
Genesis CPU
Genesis OS
Genesis Assembly Language
Genesis Hardware Board
```

This allows developers and students to understand **how computers work from transistor logic all the way to operating systems**.

---

## Contributing

Contributions are welcome.

Areas where help is needed:

* CPU microarchitecture
* ISA design
* Verilog improvements
* assembler implementation
* operating system development
* FPGA ports
* documentation

---

## License

Open source.

License will be added soon.

---

## Author

**Brilliant Narlo**

Software Engineer
Embedded Systems Engineer
Systems Programmer

GitHub:
[https://github.com/brilliantnarlo](https://github.com/brilliantnarlo)

---

If you want, I can also help you create **3 things that will make this project look extremely serious on GitHub**:

1️⃣ A **Genesis ISA specification** (`ISA.md`)
2️⃣ A **CPU instruction table**
3️⃣ A **Genesis Assembly language design**

Those three together make the project look like a **real CPU architecture project (like RISC-V or ARM)**.
