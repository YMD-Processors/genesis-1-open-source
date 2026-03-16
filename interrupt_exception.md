# What Interrupt / Exception Vectoring Means

Interrupts and exceptions force the CPU to **stop normal execution and jump to a handler**.

The Program Counter must redirect execution to a **vector address**.

Example:

```text
PC = 0x0000000000001000
Interrupt occurs
```

Execution must change to something like:

```text
PC = 0x0000000000008000   (interrupt handler)
```

This jump is called **vectoring**.

---

# Difference Between Interrupt and Exception

Although both redirect the PC, they originate differently.

| Type      | Cause                                                            |
| --------- | ---------------------------------------------------------------- |
| Interrupt | External event (timer, device, I/O)                              |
| Exception | Internal error (divide by zero, page fault, illegal instruction) |

Both require the PC to **jump to a predefined handler address**.

---

# Signals Required for Vectoring

The PC requires **three inputs**.

### Interrupt Signal

```text
interrupt
```

Meaning:

| Value | Meaning            |
| ----- | ------------------ |
| 0     | No interrupt       |
| 1     | Interrupt occurred |

---

### Exception Signal

```text
exception
```

Meaning:

| Value | Meaning            |
| ----- | ------------------ |
| 0     | No exception       |
| 1     | Exception occurred |

---

### Vector Address

```text
vector_addr
```

This is the address of the **handler routine**.

Examples:

```text
0x8000  interrupt handler
0x9000  exception handler
```

The PC **does not compute this address**, it only receives it.

---

# Required PC Behavior

When either event occurs, the PC must jump to the handler.

Logic:

```text
if interrupt
    PC = vector_addr
else if exception
    PC = vector_addr
else
    PC = normal execution
```

Interrupts and exceptions **override normal instruction flow**.

---

# Priority in PC Control

Interrupts and exceptions must have **very high priority**.

Typical PC priority order:

```text
1 reset
2 exception
3 interrupt
4 flush
5 branch
6 jump
7 sequential
```

Meaning the PC must redirect **immediately**.

---

# Example Timeline

Assume:

```text
PC = 0x100
```

Execution:

| Cycle | Event            | PC     |
| ----- | ---------------- | ------ |
| 0     | none             | 0x100  |
| 1     | none             | 0x104  |
| 2     | interrupt        | 0x8000 |
| 3     | handler executes | 0x8004 |

Normal execution resumes later via a **return-from-interrupt instruction**.

---

# Hardware Structure

The PC multiplexer must include another input.

```text
                        +------------------+
PC + 4 ---------------->|                  |
branch_target --------->|                  |
jump_addr ------------->|       MUX        |----> PC register
flush_target ---------->|                  |
vector_addr ----------->|                  |
                        +------------------+
                                ^
                                |
                         control logic
```


---

# Important CPU Requirement

When an interrupt or exception occurs, the CPU must also **save the current PC** somewhere.

Usually:

```text
EPC (Exception Program Counter)
```

Example flow:

```text
PC saved → EPC register
PC jumps → handler vector
```

The handler later executes:

```text
return_from_exception
```

which restores the PC.

(The EPC mechanism exists **outside the PC module**.)

---

# Hardware Cost

Interrupt / exception vectoring adds:

```text
1 interrupt signal
1 exception signal
1 vector address input
1 additional mux path
```

But it is essential for **operating systems and device handling**.

---

The next major PC capability in real CPUs is usually **PC Alignment Enforcement**, which ensures instruction addresses follow architectural alignment rules.
