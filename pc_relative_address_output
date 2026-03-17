In real CPUs, many instructions require addresses **relative to the current PC**.

Examples:

```
AUIPC
JAL
CALL
LOAD relative
```

These instructions use:

```
target = PC + immediate
```

So other units in the CPU must know the **current PC and sometimes PC + instruction_size**.

---

# What Your PC Is Missing

Your module only outputs:

```
pc_next
```

But the CPU normally needs:

1. **Current PC**
2. **Next sequential PC (PC + instruction size)**

Your current signal name `pc_next` is actually behaving as the **current PC register**, which can be confusing.

---

# Required Outputs for PC-Relative Addressing

The PC module should expose at least:

### Current PC

```
pc
```

Used by:

```
branch calculation
PC-relative loads
AUIPC instructions
```

Example use:

```
target = pc + offset
```

---

### Sequential PC (PC + instruction size)

```
pc_plus
```

Usually:

```
pc_plus = pc + instruction_size
```

or in your design:

```
pc_plus = pc + COUNTER
```

Used by:

```
return address generation
link registers
pipeline fetch
```

Example:

```
JAL rd,target
rd = PC + 4
```

---

# Why This Must Be an Output of the PC

Other CPU blocks depend on it:

| Unit             | Needs PC               |
| ---------------- | ---------------------- |
| Branch unit      | PC + offset            |
| Jump unit        | PC + offset            |
| Call instruction | PC + instruction_size  |
| Load/store       | PC-relative addressing |
| Debug unit       | current PC             |

If the PC module does not expose these values, other blocks must **recalculate them redundantly**.

---

# Correct Hardware Structure

Internally the PC register holds:

```
pc_reg
```

Then combinational outputs generate:

```
pc_current
pc_plus
```

Conceptually:

```
            +----------------+
clk ------> |   PC register  | ----> pc_current
            +----------------+
                     |
                     v
                 +-------+
                 | adder |
                 +-------+
                     |
                     v
                   pc_plus
```

---


# Result

Your PC now provides:

| Signal    | Meaning                     |
| --------- | --------------------------- |
| `pc`      | current instruction address |
| `pc_plus` | sequential next address     |

These allow other CPU blocks to compute:

```
pc + immediate
pc + offset
return_address = pc_plus
```

which are essential for **PC-relative instructions**.
