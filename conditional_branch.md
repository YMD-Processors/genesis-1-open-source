# What Conditional Branch Handling Means

A **conditional branch** changes the Program Counter **only if a condition is true**.

Example instruction:

```
BEQ r1, r2, label
```

Meaning:

```
if (r1 == r2)
    PC = label
else
    PC = PC + 4
```

So the PC must support **two possible next addresses**:

```
Sequential execution
Branch target
```

---

# What Signals the PC Needs

To support conditional branches the PC requires **two inputs**.

### 1. Branch Taken Signal

```
branch_taken
```

Meaning:

| Value | Meaning          |
| ----- | ---------------- |
| 0     | Branch NOT taken |
| 1     | Branch taken     |

This signal comes from the **execute stage** where the comparison happens.

Example comparison logic (outside the PC):

```
branch_taken = (r1 == r2)
```

---

### 2. Branch Target Address

```
branch_target
```

This is the address the PC should jump to **if the branch is taken**.

Example:

```
branch_target = PC + offset
```

But that calculation happens **outside the PC**.

The PC only **receives the result**.

---

# Required PC Behavior

The PC must follow this rule:

```
if branch_taken
    PC = branch_target
else
    PC = PC + 4
```

The PC itself **does not evaluate conditions**.

It only receives the decision.

---

# Updated PC Decision Logic

Considering only **conditional branching**, the PC selection becomes:

```
1 reset
2 branch_taken
3 jump
4 sequential
```

Meaning:

```
if reset
    PC = RESET_VECTOR
else if branch_taken
    PC = branch_target
else if jump
    PC = jump_addr
else
    PC = PC + 4
```

---

# Example Timing

Instruction stream:

```
0x0000  ADD
0x0004  BEQ r1,r2,0x20
0x0008  ADD
0x000C  ADD
```

Assume:

```
r1 == r2
```

Execution:

| Cycle | branch_taken | PC     |
| ----- | ------------ | ------ |
| 0     | 0            | 0x0000 |
| 1     | 0            | 0x0004 |
| 2     | 1            | 0x0020 |
| 3     | 0            | 0x0024 |

The PC **skips instructions** at `0x0008` and `0x000C`.

---

# Hardware Structure

Inside the PC unit, this is simply a **multiplexer**.

```
                +------------------+
PC + 4 -------->|                  |
                |      MUX         |----> PC register
branch_target ->|                  |
                +------------------+
                        ^
                        |
                   branch_taken
```

If `branch_taken = 1`, the mux selects **branch_target**.

---

# Important Design Principle

The **Program Counter does not compute branch conditions**.

It only receives:

```
branch_taken
branch_target
```

All comparison logic belongs to:

```
ALU / Branch Unit
```

The PC simply **redirects execution**.

---

# Minimal Hardware Cost

Conditional branching only adds:

```
1 multiplexer
1 control signal
1 new address input
```

Which is why even the earliest CPUs supported it.
