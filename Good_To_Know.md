```text
PC = 8 bits → what is the limit of instructions per line?
```

---

# First: What PC = 8-bit Means

```text
PC width = 8 bits → 2^8 = 256 addresses
```

So your CPU can point to:

```text
0x00 → 0xFF
```

That is:

```text
256 memory locations
```

---

# Now the Key Question

> “What is a single instruction per line limit?”

This depends on:

```text
Instruction size (in bytes)
```

---

# Case 1 — Instruction = 1 byte (8 bits)

```text
Each address = one instruction
```

So:

```text
256 addresses → 256 instructions
```

---

### Layout

```text
Address   Instruction
0x00      instr0
0x01      instr1
0x02      instr2
...
0xFF      instr255
```

---

# Case 2 — Instruction = 2 bytes (16 bits)

Now each instruction takes **2 addresses**:

```text
PC increments by 2
```

So:

```text
256 bytes / 2 = 128 instructions
```

---

### Layout

```text
0x00–0x01 → instr0
0x02–0x03 → instr1
...
```

---

# Case 3 — Instruction = 4 bytes (Genesis One)

```text
Instruction size = 4 bytes
```

So:

```text
256 bytes / 4 = 64 instructions
```

---

### Layout

```text
0x00–0x03 → instr0
0x04–0x07 → instr1
0x08–0x0B → instr2
...
```

---

# General Formula

```text
Number of instructions =
    (2^PC_width) / (instruction_size_in_bytes)
```

---

## For your example:

If:

```text
PC = 8 bits
Instruction = 4 bytes
```

Then:

```text
256 / 4 = 64 instructions max
```

---

# Important Clarification

```text
PC tracks ADDRESSES, not instructions
```

So:

* PC steps through memory
* Instructions occupy memory
* Size of instruction determines how many fit

---

# What “Single Line” Really Means

When you say:

```text
“one instruction per line”
```

You actually mean:

```text
one instruction per PC step
```

That only happens if:

```text
instruction size = 1 byte
```

---

# Real CPU Comparison

| CPU           | PC Width | Instr Size | Max Instructions |
| ------------- | -------- | ---------- | ---------------- |
| 8-bit CPU     | 8-bit    | 1 byte     | 256              |
| Simple 16-bit | 8-bit    | 2 bytes    | 128              |
| Your style    | 8-bit    | 4 bytes    | 64               |

---

# Final Answer

If:

```text
PC = 8 bits
Instruction = 4 bytes
```

Then:

```text
Maximum instructions = 64
```

---

# Key Insight

```text
PC width limits MEMORY
Instruction size limits HOW MANY instructions fit
```
