# **Genesis One CPU – Stall / Hold Control (Program Counter)**

## **1. Definition**

In the **Genesis One CPU**, a **stall** is a pipeline control condition that prevents the Program Counter (PC) from advancing.

> When `stall = 1`, the PC **must hold its current value**, regardless of most control-flow events.

---

## **2. Formal Behavior in Genesis One PC**

From your implementation:

```verilog
else if (stall)
    next_pc = pc_reg;
```

### **Meaning**

```text
stall = 1 → PC_next = PC_current (HOLD)
stall = 0 → PC follows arbitration logic
```

However, in **Genesis One**, stall is **NOT absolute priority**.

---

## **3. True Priority in Genesis One CPU**

Your actual arbitration logic defines:

### **Final Priority Order (Highest → Lowest)**

| Priority | Condition    | Action             |
| -------- | ------------ | ------------------ |
| 1        | Exception    | PC ← vector_addr   |
| 2        | Interrupt    | PC ← vector_addr   |
| 3        | Flush        | PC ← flush_target  |
| 4        | Stall        | PC ← PC (hold)     |
| 5        | Prediction   | PC ← predicted_pc  |
| 6        | Branch Taken | PC ← branch_target |
| 7        | Jump         | PC ← jump_addr     |
| 8        | Default      | PC ← PC + 4        |

---

## **4. Critical Insight (IMPORTANT)**

Unlike simple CPUs:

> ⚠️ **Stall in Genesis One does NOT override exceptions, interrupts, or flushes.**

### **Why this matters**

Even if the pipeline is stalled:

* An **exception MUST redirect immediately**
* An **interrupt MUST be serviced immediately**
* A **flush MUST correct the pipeline immediately**

---

## **5. Stall vs Fetch Handshake (Key Design Detail)**

Your PC update condition is:

```verilog
else if (pc_valid && fetch_ready)
    pc_reg <= next_pc;
```

And:

```verilog
pc_valid = !stall && !reset;
```

---

### **What this means**

| Signal        | Role                                       |
| ------------- | ------------------------------------------ |
| `stall`       | Freezes pipeline logically                 |
| `fetch_ready` | Backpressure from instruction memory/cache |

---

### **Combined Behavior**

Even if `next_pc` is computed:

👉 The PC **only updates when BOTH are true**:

```text
stall = 0 AND fetch_ready = 1
```

---

### **Effective Stall Conditions**

The PC will **not move** if:

* `stall = 1` ✅ (pipeline stall)
* `fetch_ready = 0` ✅ (memory not ready)

---

## **6. Real-World Stall Scenarios in Genesis One**

### **6.1 Instruction Cache Miss**

```
PC → L1 → miss → L2 → miss → L3 → memory
```

During this latency:

```text
fetch_ready = 0 → PC is frozen
```

Even if:

```text
stall = 0
```

---

### **6.2 Pipeline Hazard**

Example:

```
LOAD r1, [addr]
ADD  r2, r1, r3
```

If data is not ready:

```text
stall = 1 → PC holds
```

---

### **6.3 Structural Hazard / Resource Conflict**

* Execution unit busy
* Memory port contention

→ `stall = 1`

---

### **6.4 Branch Misprediction Recovery**

When misprediction occurs:

```text
flush = 1 → overrides stall → PC jumps to correct target
```

---

## **7. Internal Stall Behavior**

### **Next PC Logic**

```text
if stall
    next_pc = pc
```

---

### **Register Write Control (Actual Hardware Behavior)**

```text
PC updates only if:

write_enable = pc_valid && fetch_ready
```

Where:

```text
pc_valid = !stall
```

---

### **Equivalent Hardware Model**

```
                +----------------------+
PC_next ------> |   PC Register        | ----> PC
                +----------------------+
                         ^
                         |
                write_enable = !stall && fetch_ready
```

---

## **8. Timing Example (Genesis One Accurate)**

Assume:

```
PC increment = 4
```

| Cycle | stall | fetch_ready | Action          | PC     |
| ----- | ----- | ----------- | --------------- | ------ |
| 0     | 0     | 1           | Normal          | 0x0000 |
| 1     | 0     | 1           | +4              | 0x0004 |
| 2     | 1     | 1           | Stall           | 0x0004 |
| 3     | 1     | 0           | Still stalled   | 0x0004 |
| 4     | 0     | 0           | Fetch not ready | 0x0004 |
| 5     | 0     | 1           | Resume          | 0x0008 |

---

## **9. Genesis One Design Philosophy**

### **Separation of Concerns**

| Mechanism     | Purpose                      |
| ------------- | ---------------------------- |
| `stall`       | Pipeline correctness         |
| `fetch_ready` | Memory/cache synchronization |

---

### **Why This Is Powerful**

This design allows:

* High-performance pipelining
* Cache-aware execution
* Clean backpressure handling
* Precise control flow recovery

---

## **10. Key Takeaways**

* Stall **freezes PC logically**
* Fetch readiness **controls physical update**
* Stall is **NOT highest priority**
* Exceptions/interrupts/flush **override stall**
* PC update requires:

  ```text
  !stall AND fetch_ready
  ```