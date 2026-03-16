# What Pipeline Flush Means

A **pipeline flush** happens when the CPU has already fetched **incorrect instructions** and must discard them.

The PC must **redirect execution to the correct address immediately**.

Example situation:

```text
PC = 0x100
Instruction = BEQ r1,r2,target
```

While the branch is being evaluated, the CPU may already fetch:

```text
0x104
0x108
0x10C
```

If the branch is taken, those instructions are **wrong**.

So the pipeline must:

```text
1. Discard wrong instructions
2. Redirect PC to the correct address
```

That redirect is the **pipeline flush**.

---

# Signals Required for Pipeline Flush

The PC needs two signals.

### Flush Control Signal

```text
flush
```

Meaning:

| Value | Meaning                   |
| ----- | ------------------------- |
| 0     | Normal execution          |
| 1     | Pipeline must redirect PC |

---

### Flush Target Address

```text
flush_target
```

This is the **correct instruction address** to restart execution.

Examples:

* Branch misprediction correction
* Exception handler
* Interrupt entry
* Trap handler

---

# Required PC Behavior

The PC must behave like this:

```text
if flush
    PC = flush_target
else
    PC = normal_update
```

The flush must **override all normal PC updates**.

---

# Priority of Flush

Flush must have **higher priority than branches or jumps** because it corrects mistakes already made.

PC decision order:

```text
1 reset
2 flush
3 branch
4 jump
5 sequential
```


---

# Example Timeline

Assume:

```text
PC = 0x100
branch predicted NOT taken
```

Pipeline fetches:

```text
0x104
0x108
0x10C
```

Then branch resolves **taken**.

CPU asserts:

```text
flush = 1
flush_target = 0x200
```

Timeline:

| Cycle | flush | PC    |
| ----- | ----- | ----- |
| 0     | 0     | 0x100 |
| 1     | 0     | 0x104 |
| 2     | 0     | 0x108 |
| 3     | 1     | 0x200 |

All instructions after `0x100` are discarded.

---

# Hardware Structure

The PC must include another **input to the PC selection multiplexer**.

```text
                     +------------------+
PC + 4  ------------>|                  |
branch_target ------>|                  |
jump_addr ---------->|       MUX        |----> PC register
flush_target ------->|                  |
                     +------------------+
                             ^
                             |
                       control logic
```

When `flush = 1`, the mux selects **flush_target**.

---

# Important Property of Flush

Flush is **not just for branches**.

It is used for **all pipeline recovery events**:

Examples:

```text
branch misprediction
exception
interrupt
illegal instruction
memory fault
```

The PC must **immediately redirect execution**.

---

# Hardware Cost

Pipeline flush support requires only:

```text
1 additional control signal
1 additional address input
1 additional mux input
```

But it is **critical for modern pipelined CPUs**.
