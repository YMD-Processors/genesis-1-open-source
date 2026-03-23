# **Genesis One CPU – Program Counter (PC) Module**

### **Module Name:** `pc`

### **File:** `pc.v`

### **Category:** Instruction Fetch Unit (IFU)

---

Abstract

This document specifies the architecture, behavior, and interface of the Program Counter (PC) module for the Genesis One CPU. The PC module is responsible for maintaining the current instruction address and determining the next instruction address based on control flow events such as sequential execution, branching, jumping, exceptions, interrupts, and speculative execution via branch prediction.

---

## **1. Overview**

The **Program Counter (PC)** module is responsible for generating and maintaining the current instruction address in the Genesis One CPU pipeline.

It supports:

* Sequential instruction flow
* Branching and jumping
* Interrupt and exception handling
* Pipeline control (stall, flush)
* Branch prediction integration

The PC updates synchronously with the system clock and outputs both the current instruction address and the next sequential address.

---

## **2. Key Features**

* 64-bit configurable architecture (`XLEN`)
* Instruction alignment (4-byte aligned)
* Multi-source next-PC arbitration
* Integrated branch prediction support
* Interrupt and exception vector handling
* Pipeline stall and flush control
* Fetch-stage handshake (`fetch_ready`)

---

## **3. Parameters**

| Parameter | Width       | Description                            |
| --------- | ----------- | -------------------------------------- |
| `XLEN`    | Default: 64 | Width of the program counter           |
| `COUNTER` | Default: 4  | Instruction step size (byte increment) |

---

## **4. Port Description**

### **Inputs**

| Signal             | Width | Description                         |
| ------------------ | ----- | ----------------------------------- |
| `clk`              | 1     | System clock                        |
| `reset`            | 1     | Asynchronous reset                  |
| `stall`            | 1     | Pipeline stall signal               |
| `reset_vector`     | XLEN  | Reset start address                 |
| `branch_taken`     | 1     | Indicates a taken branch            |
| `branch_target`    | XLEN  | Branch destination address          |
| `flush`            | 1     | Pipeline flush signal               |
| `flush_target`     | XLEN  | Flush redirection address           |
| `interrupt`        | 1     | Interrupt request                   |
| `exception`        | 1     | Exception request                   |
| `vector_addr`      | XLEN  | Interrupt/exception handler address |
| `prediction_valid` | 1     | Branch prediction valid             |
| `predicted_pc`     | XLEN  | Predicted next PC                   |
| `jump`             | 1     | Jump instruction signal             |
| `jump_addr`        | XLEN  | Jump target address                 |
| `fetch_ready`      | 1     | Instruction fetch readiness         |

---

### **Outputs**

| Signal     | Width | Description                   |
| ---------- | ----- | ----------------------------- |
| `pc`       | XLEN  | Current program counter       |
| `pc_plus`  | XLEN  | Next sequential PC (`pc + 4`) |
| `pc_valid` | 1     | Indicates valid PC output     |

---

## **5. Functional Description**

### **5.1 PC Register**

The core register holds the current instruction address:

```verilog
reg [XLEN-1:0] pc_reg;
```

* Updated on rising clock edge
* Reset asynchronously to `reset_vector`
* Only updates when:

  * `pc_valid = 1`
  * `fetch_ready = 1`

---

### **5.2 Sequential Execution**

Default PC progression:

```
PC_next = PC + 4
```

Controlled by:

```verilog
pc_plus = pc_reg + COUNTER;
```

---

### **5.3 Address Alignment**

All control-flow addresses are aligned to **4-byte boundaries**:

```verilog
{address[XLEN-1:2], 2'b00}
```

#### Purpose:

* Ensures instruction alignment
* Prevents misaligned instruction fetch
* Matches typical ISA requirements (e.g., RISC-style)

---

### **5.4 Next PC Arbitration Logic**

The PC selection logic follows a strict **priority order**:

### **Priority Table (Highest → Lowest)**

| Priority | Condition  | Next PC Source  |
| -------- | ---------- | --------------- |
| 1        | Exception  | `vector_addr`   |
| 2        | Interrupt  | `vector_addr`   |
| 3        | Flush      | `flush_target`  |
| 4        | Stall      | Hold current PC |
| 5        | Prediction | `predicted_pc`  |
| 6        | Branch     | `branch_target` |
| 7        | Jump       | `jump_addr`     |
| 8        | Default    | `pc + 4`        |

---

### **5.5 Pipeline Control**

#### **Stall Behavior**

* PC does **not advance**
* Maintains current instruction address

```verilog
else if (stall)
    next_pc = pc_reg;
```

---

#### **Flush Behavior**

* Redirects execution immediately
* Used after misprediction or hazards

---

### **5.6 Branch Prediction Support**

When `prediction_valid` is asserted:

```verilog
next_pc = aligned_predicted_pc;
```

* Allows speculative execution
* Improves pipeline performance

---

### **5.7 Interrupt & Exception Handling**

Both use the same vector:

```verilog
next_pc = aligned_vector_addr;
```

#### Notes:

* Highest priority in control flow
* Ensures immediate redirection to handler

---

### **5.8 Fetch Handshake**

PC updates only when:

```verilog
pc_valid && fetch_ready
```

#### Purpose:

* Prevents instruction fetch overflow
* Synchronizes PC with instruction memory/cache

---

## **6. Timing Behavior**

### **PC Update Condition**

```verilog
always @(posedge clk or posedge reset)
```

| Condition                 | Action              |
| ------------------------- | ------------------- |
| `reset = 1`               | Load `reset_vector` |
| `pc_valid && fetch_ready` | Load `next_pc`      |
| Otherwise                 | Hold current PC     |

---

## **7. Output Signals**

### **pc**

* Current instruction address
* Used by instruction fetch stage

### **pc_plus**

* Precomputed next sequential address
* Used for:

  * Branch calculations
  * Return addresses

### **pc_valid**

```verilog
!stall && !reset
```

Indicates:

* PC is usable
* Pipeline is active

---

## **8. Design Considerations**

### **Alignment Enforcement**

* All addresses forced to 4-byte boundary
* Hardware-level safety against misalignment

---

### **Deterministic Priority**

* Prevents ambiguous control flow
* Guarantees predictable execution

---

### **Decoupled Fetch**

* `fetch_ready` enables backpressure handling
* Supports cache/memory latency

---

### **Scalability**

* Parameterized for:

  * 32-bit / 64-bit architectures
  * Variable instruction widths

---

## **9. Integration in Pipeline**

```
PC → L1 Instruction Cache → Decode → Execute
```

### Responsibilities:

* Supplies instruction address
* Responds to pipeline control signals
* Interfaces with branch predictor

---

## **10. Example Execution Scenarios**

### **Normal Execution**

```
PC = PC + 4
```

---

### **Branch Taken**

```
PC = branch_target
```

---

### **Branch Prediction**

```
PC = predicted_pc
```

---

### **Exception Occurs**

```
PC = vector_addr
```

---

### **Pipeline Stall**

```
PC = PC (unchanged)
```