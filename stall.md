# What Stall / Hold Control Means for a Program Counter

A **stall** means:

> The CPU pipeline cannot proceed this cycle, so the **Program Counter must NOT change**.

In other words:

```text
If stall = 1 → PC stays the same
If stall = 0 → PC updates normally
```

This is critical because instruction fetch must stay synchronized with the rest of the pipeline.

---

# Why Stall Control Is Needed

The PC must stop advancing when the processor cannot safely fetch the next instruction.

Common reasons include:

### Instruction Cache Miss

The CPU requested an instruction but memory has not returned it yet.

```
cycle 1: request instruction
cycle 2-20: memory waiting
```

During this time the PC **must remain frozen**.

---

### Pipeline Hazard

Example:

```
ADD r1,r2,r3
SUB r4,r1,r5
```

The second instruction needs the result of the first.
If forwarding cannot resolve it, the pipeline inserts a stall.

PC must **not advance** or the pipeline will desynchronize.

---

### Memory Wait States

Slow RAM or peripherals can delay the pipeline.

Again the PC must stay fixed.

---

# What Stall Does to the PC Internally

When a stall occurs, the register **does not update**.

Normal PC logic:

```
PC_next = PC + 4
```

With stall:

```
if stall
    PC_next = PC
else
    PC_next = normal_update
```

---

# Required Signal

Add **one signal only**:

```
stall
```

Definition:

| Signal    | Meaning                 |
| --------- | ----------------------- |
| stall = 1 | Hold PC (do not update) |
| stall = 0 | PC updates normally     |

---

# Correct PC Update Priority (Only Considering Stall)

For a PC with **stall support**, the priority becomes:

```
1 reset
2 stall
3 jump
4 sequential increment
```

Meaning:

```
if reset
    PC = RESET_VECTOR
else if stall
    PC = PC
else if jump
    PC = jump_addr
else
    PC = PC + 4
```

---


# Timing Example

Assume:

```
PC increment = 4
```

Clock timeline:

| Cycle | stall | PC     |
| ----- | ----- | ------ |
| 0     | 0     | 0x0000 |
| 1     | 0     | 0x0004 |
| 2     | 1     | 0x0004 |
| 3     | 1     | 0x0004 |
| 4     | 0     | 0x0008 |

While **stall = 1**, the PC **freezes**.

---

# Hardware Implementation

In hardware the stall simply **disables the register write**.

Conceptually:

```
          +----------------+
PC -----> |   PC Register  | ----> PC_out
          +----------------+
                 ^
                 |
            write_enable
                 ^
                 |
        !(stall)
```

If `stall = 1`, the register **does not load a new value**.

---

# Important Design Note

You already have something **very close to stall**:

```
pc_en
```

But in real CPUs:

```
pc_en = pipeline running
stall = pipeline stopped
```

Using an explicit **stall signal** is clearer and safer in complex designs.

---
