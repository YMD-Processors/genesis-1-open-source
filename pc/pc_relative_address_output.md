# **Genesis One CPU – Program Counter (PC) Module (Updated)**

### **Module Name:** `pc`

### **Category:** Instruction Fetch Unit (IFU)

### **Feature Update:** PC-Relative Addressing Support

---

## **1. Overview**

The **Program Counter (PC)** module generates and maintains the current instruction address and provides critical outputs required for **PC-relative instruction execution**.

Modern ISAs rely heavily on **PC-relative addressing**, where target addresses are computed as:

```
target = PC + immediate
```

To support this efficiently, the Genesis One PC module explicitly exposes:

* The **current PC**
* The **sequential PC (PC + instruction size)**

This eliminates redundant calculations across execution units and improves pipeline efficiency.

---

## **2. Key Enhancement: PC-Relative Addressing Support**

### **Why This Matters**

Many instructions depend on the PC:

```
AUIPC
JAL / CALL
PC-relative LOAD/STORE
Branches
```

All of these require:

```
target = PC + offset
```

---

## **3. Correct PC Signal Semantics**

### ✅ **Clarified Naming**

| Signal    | Meaning                               |
| --------- | ------------------------------------- |
| `pc`      | **Current PC (pc_reg)**               |
| `pc_plus` | **Next sequential PC (pc + COUNTER)** |

> ⚠️ Note: In earlier interpretations, `pc_next` could be confused with "next PC".
> In Genesis One, `pc` is the **true current PC**, and `pc_plus` is the **sequential next address**.

---

## **4. Outputs for PC-Relative Addressing**

### **4.1 Current PC**

```verilog
output wire [XLEN-1:0] pc;
```

### **Purpose**

Used by execution units to compute:

```
target = pc + immediate
```

### **Consumers**

| Unit            | Usage                  |
| --------------- | ---------------------- |
| Branch Unit     | `pc + offset`          |
| ALU             | AUIPC calculations     |
| Load/Store Unit | PC-relative addressing |
| Debug Unit      | Instruction tracing    |

---

### **4.2 Sequential PC (PC + Instruction Size)**

```verilog
output wire [XLEN-1:0] pc_plus;
```

### **Definition**

```verilog
pc_plus = pc + COUNTER;
```

### **Purpose**

Used for:

```
return_address = PC + instruction_size
```

### **Consumers**

| Unit                  | Usage               |
| --------------------- | ------------------- |
| Jump/Link (JAL)       | Save return address |
| Call instructions     | Link register       |
| Pipeline Fetch        | Next instruction    |
| Speculative execution | Prediction fallback |

---

## **5. Internal Architecture**

### **5.1 PC Register**

```verilog
reg [XLEN-1:0] pc_reg;
```

This holds the **current instruction address**.

---

### **5.2 Output Generation**

```verilog
assign pc      = pc_reg;
assign pc_plus = pc_reg + COUNTER;
```

---

## **6. Hardware Structure (Conceptual)**

```
            +----------------------+
clk ------> |     PC Register      | ----> pc (current)
            |      (pc_reg)        |
            +----------------------+
                       |
                       v
                 +------------+
                 |   Adder    |
                 | +COUNTER   |
                 +------------+
                       |
                       v
                    pc_plus
```

---

## **7. PC-Relative Address Computation**

### **General Formula**

```
target = pc + immediate
```

---

### **Example 1: AUIPC**

```
rd = pc + immediate
```

---

### **Example 2: JAL (Jump and Link)**

```
rd = pc_plus
pc = pc + offset
```

---

### **Example 3: Branch**

```
if (condition)
    pc = pc + offset
```

---

### **Example 4: PC-Relative Load**

```
address = pc + offset
load(address)
```

---

## **8. Why PC Must Provide These Outputs**

Without these outputs, other units would need to recompute:

```
pc + immediate
pc + 4
```

This leads to:

* Redundant adders
* Increased hardware cost
* Timing inconsistencies

---

### **Centralized PC Advantages**

| Benefit      | Description               |
| ------------ | ------------------------- |
| Efficiency   | Single adder reused       |
| Consistency  | Same PC base across units |
| Timing       | Reduced critical paths    |
| Clean design | Better modularity         |

---

## **9. Pipeline Integration Impact**

### **Before (Incomplete Design)**

```
PC → Fetch
(No direct PC-relative support)
```

---

### **After (Correct Design)**

```
                +----------------+
                |       PC       |
                | pc, pc_plus    |
                +----------------+
                   |        |
         ----------         ----------
        v                             v
 Branch Unit                Execute / ALU
 (pc + offset)              (AUIPC, LOAD)
```

---

## **10. Updated Signal Summary**

| Signal     | Type   | Description                 |
| ---------- | ------ | --------------------------- |
| `pc`       | Output | Current instruction address |
| `pc_plus`  | Output | Sequential next PC          |
| `pc_valid` | Output | Valid PC indicator          |

---

## **11. Design Compliance**

This implementation aligns with modern CPU design principles:

* RISC-style PC-relative execution
* Efficient pipeline data sharing
* Minimal redundant arithmetic
* Clean separation of concerns
