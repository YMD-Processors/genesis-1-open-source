# **Genesis One CPU – Program Counter (PC)**

### **Speculative Execution Interface Update**

---

## **1. Overview**

The **Program Counter (PC)** in the Genesis One CPU supports **speculative execution** through an external prediction interface.

Speculative execution allows the processor to **continue fetching instructions before branch resolution**, preventing pipeline stalls and improving performance.

The PC itself **does not perform prediction**. Instead, it provides a clean interface for external prediction units.

---

## **2. Speculative Execution Concept**

In a pipelined CPU, branch resolution occurs several cycles after fetch. Waiting for resolution introduces stalls.

To avoid this, the CPU **predicts the next PC**:

```
Sequential Flow:
0x100 → 0x104 → 0x108

With speculation:
PC jumps early to predicted target
```

This allows the pipeline to remain **fully utilized**.

---

## **3. Speculative Execution Interface**

The Genesis One PC exposes a minimal and efficient prediction interface:

### **Prediction Signals**

| Signal             | Width | Description                        |
| ------------------ | ----- | ---------------------------------- |
| `prediction_valid` | 1     | Indicates prediction is valid      |
| `predicted_pc`     | XLEN  | Predicted next instruction address |

---

### **Behavior**

```
if (prediction_valid)
    PC = predicted_pc
else
    PC = normal_flow
```

* Prediction is **external**
* PC acts as a **consumer of predicted addresses**

---

## **4. Integration in PC Arbitration Logic**

The speculative path is integrated into the PC’s priority-based selection logic.

### **Updated Priority Table**

| Priority | Condition                      | Next PC Source  |
| -------- | ------------------------------ | --------------- |
| 1        | Exception                      | `vector_addr`   |
| 2        | Interrupt                      | `vector_addr`   |
| 3        | Flush (misprediction recovery) | `flush_target`  |
| 4        | Stall                          | Hold current PC |
| 5        | **Prediction (Speculative)**   | `predicted_pc`  |
| 6        | Branch Taken                   | `branch_target` |
| 7        | Jump                           | `jump_addr`     |
| 8        | Default                        | `pc + 4`        |

---

### **Key Insight**

Speculation occurs **before branch resolution**:

* Prediction has **higher priority than branch_taken**
* Enables early redirection of instruction fetch

---

## **5. Prediction Flow**

### **Example Instruction Stream**

```
0x100  BEQ r1, r2, target
```

### **Prediction Case: Branch Taken**

| Cycle | prediction_valid | PC Value   |
| ----- | ---------------- | ---------- |
| 0     | 0                | 0x100      |
| 1     | 1                | target     |
| 2     | 0                | target + 4 |

The CPU immediately begins executing the predicted path.

---

## **6. Misprediction Handling**

If a prediction is incorrect, the pipeline must recover using **flush control**.

### **Mechanism**

```
flush = 1
flush_target = correct_address
```

### **PC Reaction**

* Flush overrides prediction
* PC is redirected to the correct address

---

### **Priority Relationship**

```
flush > prediction
```

This guarantees:

* Correct execution state
* Safe recovery from incorrect speculation

---

## **7. Hardware Architecture**

The speculative execution interface adds one additional input path to the PC selection logic.

```
                      +----------------------+
PC + COUNTER -------->|                      |
branch_target -------->|                      |
jump_addr ------------>|                      |
flush_target --------->|        PC MUX        |----> PC register
vector_addr ---------->|                      |
predicted_pc --------->|                      |
                      +----------------------+
                               ^
                               |
                      prediction_valid
```

---

## **8. Design Philosophy**

### **Separation of Concerns**

The PC is intentionally kept simple:

* ❌ No prediction logic inside PC
* ✅ Only consumes predicted addresses

Prediction is handled by dedicated units:

* Branch Predictor
* Branch Target Buffer (BTB)
* Return Address Stack (RAS)

---

### **Why This Matters**

* Keeps PC fast and deterministic
* Allows independent evolution of predictor
* Reduces critical path complexity

---

## **9. Hardware Cost**

The speculative execution interface adds minimal overhead:

* 1 × `predicted_pc` input bus
* 1 × `prediction_valid` control signal
* 1 additional MUX input path

All complex logic resides in external predictor units.

---

## **10. Pipeline Interaction**

### **Fetch Stage Behavior**

```
IF Stage receives:
    pc → instruction cache

PC receives:
    predicted_pc from predictor
```

---

### **Execution Feedback Loop**

```
Execute Stage → branch result
                ↓
          Mispredict?
                ↓
           Flush signal
                ↓
               PC
```

---

## **11. Correctness Guarantees**

The design ensures:

* Speculation never commits incorrect state
* All incorrect paths are discarded via flush
* Highest-priority events (exceptions, interrupts) override speculation

---

## **12. Summary**

The Genesis One PC speculative execution interface enables:

```
PC = predicted_pc   (when prediction_valid = 1)
```

This allows:

* Continuous instruction fetch
* Reduced pipeline stalls
* Higher instruction throughput