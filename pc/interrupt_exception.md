# **Genesis One CPU – Interrupt & Exception Vectoring (PC Module)**

---

## **1. Overview**

In the **Genesis One CPU**, interrupts and exceptions are **high-priority control-flow events** that force the Program Counter (PC) to immediately redirect execution to a handler.

This mechanism is called **vectoring**.

### **Definition**

Vectoring is the process of:

```text
Redirecting PC → predefined handler address (vector_addr)
```

---

## **2. Genesis One Behavior**

From your PC design:

```verilog
if (exception)
    next_pc = aligned_vector_addr;
else if (interrupt)
    next_pc = aligned_vector_addr;
```

### **Key Insight**

* Both **interrupts and exceptions share the same vector input**
* **Exception has higher priority than interrupt**
* Address is **hardware-aligned to 4 bytes**

---

## **3. Interrupt vs Exception (Genesis One Context)**

| Type      | Source             | Priority | PC Action             |
| --------- | ------------------ | -------- | --------------------- |
| Exception | Internal CPU fault | Higher   | Jump to `vector_addr` |
| Interrupt | External event     | Lower    | Jump to `vector_addr` |

---

## **4. Required Signals**

### **4.1 Interrupt Signal**

```text
interrupt
```

| Value | Meaning                    |
| ----- | -------------------------- |
| 0     | No interrupt               |
| 1     | External interrupt request |

---

### **4.2 Exception Signal**

```text
exception
```

| Value | Meaning                 |
| ----- | ----------------------- |
| 0     | No exception            |
| 1     | Internal fault detected |

---

### **4.3 Vector Address**

```text
vector_addr
```

* Input to PC module
* Already computed by:

  * Control Unit / CSR Unit (future)
  * Exception/Interrupt controller

---

## **5. Address Alignment (Critical Detail)**

Genesis One enforces **instruction alignment in hardware**:

```verilog
aligned_vector_addr = {vector_addr[XLEN-1:2], 2'b00};
```

### **Implications**

* All handlers are **4-byte aligned**
* Prevents illegal instruction fetch
* Guarantees ISA compliance

---

## **6. Next-PC Arbitration (Updated for Genesis One)**

### **Actual Priority Order (From Your Code)**

```text
1 Exception
2 Interrupt
3 Flush
4 Stall
5 Prediction
6 Branch
7 Jump
8 Sequential (PC + 4)
```

### **Important Difference from Generic CPUs**

* **Stall is above prediction/branch/jump**
* **Prediction occurs before branch resolution**
* Interrupt/Exception override EVERYTHING except reset

---

## **7. Functional Logic**

### **Exact Behavior**

```text
if exception:
    PC = vector_addr
else if interrupt:
    PC = vector_addr
else:
    PC = normal flow
```

### **With Alignment Applied**

```text
PC = {vector_addr[XLEN-1:2], 00}
```

---

## **8. Timing Behavior**

PC update only occurs when:

```text
pc_valid && fetch_ready
```

### **Important Implication**

Even interrupts/exceptions:

* **Wait for fetch stage readiness**
* Prevents pipeline corruption
* Ensures safe redirection

---

## **9. Example Execution Timeline (Genesis One)**

Assume:

```text
PC = 0x0000000000000100
vector_addr = 0x0000000000008003   (unaligned input)
```

### **Hardware Alignment**

```text
aligned_vector_addr = 0x0000000000008000
```

---

### **Execution**

| Cycle | Event              | PC     |
| ----- | ------------------ | ------ |
| 0     | Normal             | 0x100  |
| 1     | Normal             | 0x104  |
| 2     | Interrupt asserted | 0x8000 |
| 3     | Handler executes   | 0x8004 |

---

## **10. Hardware Structure (Genesis One PC MUX)**

The PC uses a prioritized combinational selector:

```text
                        +----------------------+
PC + 4 ---------------->|                      |
branch_target --------->|                      |
jump_addr ------------->|                      |
flush_target ---------->|      PC MUX          |----> PC Register
predicted_pc ---------->|                      |
vector_addr ----------->|                      |
                        +----------------------+
                                ^
                                |
                        Priority Control Logic
```

### **Control Logic Includes**

* exception
* interrupt
* flush
* stall
* prediction_valid
* branch_taken
* jump

---

## **11. Pipeline Interaction**

### **Flush vs Interrupt/Exception**

Even if a flush occurs:

```text
exception/interrupt OVERRIDE flush
```

---

### **Stall Interaction**

If `stall = 1`:

```text
PC holds value UNLESS interrupt/exception occurs
```

⚠️ **But in your implementation:**

```verilog
stall comes AFTER flush but BEFORE prediction
```

Meaning:

* Interrupt/Exception still win
* Stall only blocks lower-priority events

---

## **12. EPC Requirement (Outside PC Module)**

Genesis One requires an external register:

### **EPC (Exception Program Counter)**

Before vectoring:

```text
EPC ← current PC
PC  ← vector_addr
```

### **Return Mechanism**

Handled by instruction like:

```text
RET / ERET (future ISA)
```

Which performs:

```text
PC ← EPC
```

---

## **13. Design Implications**

### **Why This Design is Strong**

* Deterministic priority (no ambiguity)
* Fast redirection (single-cycle decision)
* Alignment-safe
* Pipeline-safe (fetch_ready gating)

---

### **Scalability**

Future extensions:

* Separate interrupt/exception vectors
* Vector table indexing
* Privilege modes (user/kernel)
* CSR-based trap handling

---

## **14. Hardware Cost (Genesis One)**

| Component          | Cost    |
| ------------------ | ------- |
| Interrupt signal   | +1      |
| Exception signal   | +1      |
| Vector address bus | +XLEN   |
| MUX input          | +1 path |
| Control logic      | Minimal |

