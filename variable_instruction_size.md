# What Variable Instruction Size Means

In some CPU architectures, **instructions are not all the same size**.

Instead of always advancing:

```text
PC = PC + 4
```

the PC must advance by **the length of the instruction that was just executed**.

Example instruction sizes:

```text
2 bytes
4 bytes
6 bytes
8 bytes
```

The PC must therefore compute:

```text
PC_next = PC + instruction_length
```

---

# Why Some CPUs Use Variable Instruction Length

Variable-length instructions allow:

1. **Better code density** (smaller programs)
2. **More complex instructions**
3. **Backward compatibility**

Examples:

| CPU       | Instruction Size |
| --------- | ---------------- |
| x86       | 1–15 bytes       |
| RISC-V    | 2 or 4 bytes     |
| ARM Thumb | 2 or 4 bytes     |
| MIPS      | fixed 4 bytes    |

---

# What the Program Counter Must Support

The PC must **no longer assume a fixed increment**.

Instead it must receive the **next sequential address** from the instruction decode or fetch stage.

So instead of this:

```text
PC = PC + 4
```

the PC must do:

```text
PC = PC + instruction_size
```

---

# Required Signal

The PC must receive an input representing the instruction size.

Example signal:

```text
instr_size
```

Example values:

| Value | Meaning            |
| ----- | ------------------ |
| 2     | 2-byte instruction |
| 4     | 4-byte instruction |
| 8     | 8-byte instruction |

Then sequential update becomes:

```text
PC_next = PC + instr_size
```

---

# Example Execution

Instruction stream:

```text
Address    Instruction Size
0x1000     2 bytes
0x1002     4 bytes
0x1006     2 bytes
0x1008     4 bytes
```

PC sequence becomes:

```text
0x1000
0x1002
0x1006
0x1008
0x100C
```

The PC increments by **different amounts each cycle**.

---

# Hardware Structure

Instead of a fixed adder:

```text
PC + 4
```

the PC uses a **configurable adder**.

Conceptually:

```text
          +-------------------+
PC -----> |                   |
          |       ADDER       | ----> next_pc
size ---> |                   |
          +-------------------+
```

Where:

```text
next_pc = PC + instruction_size
```

---

# Alternative Design Used in Real CPUs

Many real CPUs **do not give the PC the instruction size directly**.

Instead they compute the **next PC outside the PC module**.

Example:

```text
next_pc = PC + instruction_size
```

Then the PC simply loads:

```text
PC = next_pc
```

This simplifies the PC unit.


---

# Important Design Constraint

When variable instruction sizes are used, **alignment rules must match instruction sizes**.

Example:

If instruction sizes are:

```text
2 bytes
4 bytes
```

Then valid PC alignment becomes:

```text
PC[0] = 0
```

Instead of:

```text
PC[1:0] = 00
```

---

# Hardware Cost

Variable instruction size support adds only:

```text
1 instruction_size input
1 configurable adder
```

But it requires the **instruction decoder to determine instruction length** before the PC advances.

---

# Summary

With variable instruction sizes, the PC sequential update becomes:

```text
PC_next = PC + instruction_length
```

Instead of:

```text
PC_next = PC + constant
```

This allows CPUs to support **mixed instruction sizes within the same program**.
