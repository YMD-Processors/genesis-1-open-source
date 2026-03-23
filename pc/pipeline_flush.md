# **Genesis One CPU – Pipeline Flush Mechanism**

## **1. Overview**

A **pipeline flush** in the Genesis One CPU occurs when the processor detects that instructions currently in the pipeline are **invalid and must be discarded**.

This typically happens due to:

* Branch misprediction
* Exceptions
* Interrupt redirection
* Hazard recovery

The Program Counter (PC) must **immediately redirect execution** to a correct address using the **flush mechanism**.

---

## **2. Conceptual Explanation**

### **What Pipeline Flush Means**

When the CPU speculatively fetches instructions, it may fetch **incorrect instructions** before the actual control decision is resolved.

### Example:

```text
PC = 0x100
Instruction = BEQ r1, r2, target
```

While evaluating the branch, the pipeline continues fetching:

```text
0x104
0x108
0x10C
```

If the branch is **taken**, these instructions are invalid.

### Therefore:

```text
1. Discard incorrect instructions
2. Redirect execution to correct target
```

This redirection is called a **pipeline flush**.

---

## **3. Flush Signals in Genesis One PC**

The PC module includes dedicated flush control signals:

### **3.1 Flush Control**

```text
flush
```

| Value | Meaning                    |
| ----- | -------------------------- |
| 0     | Normal execution           |
| 1     | Pipeline redirect required |

---

### **3.2 Flush Target Address**

```text
flush_target
```

* The **correct instruction address** after recovery
* Internally aligned to 4-byte boundary:

```verilog
aligned_flush = {flush_target[XLEN-1:2], 2'b00};
```

---

## **4. PC Behavior with Flush**

In Genesis One, flush is integrated into the **Next PC Arbitration Logic**:

```verilog
else if (flush)
    next_pc = aligned_flush;
```

### **Effective Behavior**

```text
if exception → PC = vector_addr
else if interrupt → PC = vector_addr
else if flush → PC = flush_target
else → normal flow
```

---

## **5. Flush Priority in Genesis One**

Unlike simplified models, Genesis One uses a **real CPU-grade priority system**:

### **Final Priority Order (Highest → Lowest)**

```text
1. Exception
2. Interrupt
3. Flush
4. Stall (hold PC)
5. Branch Prediction
6. Branch Taken
7. Jump
8. Sequential (PC + 4)
```

---

### **Key Insight**

* **Flush overrides branch and jump**
* But **exception/interrupt override flush**

This ensures:

* Correct recovery from faults first
* Then pipeline correction

---

## **6. Pipeline Timing Example**

### Scenario: Branch Misprediction

Initial state:

```text
PC = 0x100
Prediction = NOT taken
```

Pipeline fetches:

```text
0x104
0x108
0x10C
```

Branch resolves **taken**, CPU asserts:

```text
flush = 1
flush_target = 0x200
```

---

### **Execution Timeline**

| Cycle | flush | PC    | Description   |
| ----- | ----- | ----- | ------------- |
| 0     | 0     | 0x100 | Fetch branch  |
| 1     | 0     | 0x104 | Speculative   |
| 2     | 0     | 0x108 | Speculative   |
| 3     | 1     | 0x200 | Flush applied |

---

### Result:

* Instructions at `0x104`, `0x108`, `0x10C` are **discarded**
* Execution resumes at **correct target**

---

## **7. Hardware Implementation**

### **7.1 PC MUX Structure**

Genesis One PC includes flush as a **dedicated MUX input**:

```text
                  +----------------------+
PC + 4 ---------->|                      |
branch_target ---->|                      |
jump_addr -------->|                      |
predicted_pc ----->|      PC SELECT MUX   |----> pc_reg
flush_target ----->|                      |
vector_addr ------>|                      |
                  +----------------------+
                           ^
                           |
                    arbitration logic
```

---

### **7.2 Control Logic**

Flush selection condition:

```verilog
else if (flush)
    next_pc = aligned_flush;
```

---

## **8. Interaction with Pipeline Control**

### **Flush vs Stall**

| Condition   | Behavior    |
| ----------- | ----------- |
| `flush = 1` | Redirect PC |
| `stall = 1` | Hold PC     |

### **Important Rule**

Flush has **higher priority than stall**:

```verilog
else if (flush)
    next_pc = aligned_flush;

else if (stall)
    next_pc = pc_reg;
```

---

## **9. Interaction with Fetch Stage**

PC updates only when:

```verilog
pc_valid && fetch_ready
```

### Implication:

Even during flush:

* PC waits for `fetch_ready`
* Prevents invalid memory access
* Ensures clean pipeline restart

---

## **10. Flush Use Cases in Genesis One**

Flush is used for **all pipeline recovery events**:

### **10.1 Branch Misprediction**

* Incorrect speculative path
* Redirect to correct branch target

---

### **10.2 Exceptions**

* Illegal instruction
* Divide by zero
* Memory faults

*(Handled via higher-priority vector, not flush directly)*

---

### **10.3 Interrupts**

* External or timer interrupts
* Redirect to handler

*(Also higher priority than flush)*

---

### **10.4 Hazard Recovery**

* Control hazards
* Speculative execution rollback

---

## **11. Design Properties**

### **Deterministic Recovery**

* Single-cycle redirect decision
* No ambiguity in control flow

---

### **Minimal Hardware Cost**

Flush support requires:

```text
+1 control signal (flush)
+1 address input (flush_target)
+1 MUX input
```

---

### **High Impact Feature**

Despite low cost, flush is critical for:

* Pipeline correctness
* Speculative execution
* High-performance CPU design