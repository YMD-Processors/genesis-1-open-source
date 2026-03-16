# What Alignment Enforcement Means

Instruction addresses in most CPU architectures must be **aligned to specific byte boundaries**.

This means the Program Counter must only generate addresses that are **valid instruction boundaries**.

Example for a 4-byte instruction architecture:

```text
Valid PC values
0x0000
0x0004
0x0008
0x000C
```

Invalid values:

```text
0x0001
0x0002
0x0003
0x0005
```

Alignment enforcement ensures the PC **never produces illegal instruction addresses**.

---

# Why Alignment Exists

CPUs align instructions because it simplifies hardware:

1. Faster instruction fetch
2. Simpler instruction decoding
3. Efficient memory access
4. Reduced control complexity

Without alignment rules, instruction fetch hardware becomes significantly more complex.

---

# Typical Alignment Rules

| Architecture | Instruction Alignment          |
| ------------ | ------------------------------ |
| MIPS         | 4 bytes                        |
| ARM          | 4 bytes                        |
| RISC-V       | 4 bytes (or 2 with compressed) |
| x86          | 1 byte (no strict alignment)   |

For most RISC architectures:

```text
alignment = 4 bytes
```

---

# What the Program Counter Must Enforce

The PC must ensure that **all next addresses respect the alignment rule**.

This applies to:

* sequential increments
* branch targets
* jump targets
* interrupt vectors
* exception vectors

All must land on valid instruction boundaries.

---

# Alignment Rule

For 4-byte instructions:

```text
PC[1:0] must equal 00
```

Meaning the two least significant bits must be zero.

Examples:

| Address | Valid |
| ------- | ----- |
| 0x1000  | ✔     |
| 0x1004  | ✔     |
| 0x1008  | ✔     |
| 0x1002  | ✖     |

---

# Two Ways CPUs Enforce Alignment

## 1. Hardware Masking

The PC **forces alignment automatically** by clearing lower bits.

Example:

```text
aligned_addr = addr & 0xFFFFFFFFFFFFFFFC
```

This guarantees the PC always points to a valid instruction boundary.

---

## 2. Alignment Exception

Instead of masking, the CPU detects misalignment and raises an **exception**.

Example:

```text
if addr[1:0] != 00
    raise instruction_address_misaligned
```

Many modern CPUs choose this method.

---

# Alignment in Sequential Execution

Your PC increments by:

```text
COUNTER = 4
```

This automatically maintains alignment:

```text
PC = PC + 4
```

But alignment must still be enforced for **external addresses** such as:

* branch_target
* jump_addr
* vector_addr

---

# Hardware Structure

Alignment enforcement typically occurs **before writing the PC register**.

Conceptually:

```text
incoming_address
       |
       v
+----------------------+
|  Alignment Logic     |
| clear lower bits     |
+----------------------+
       |
       v
    PC register
```

---

# Alignment Mask Example

For 4-byte alignment:

```text
aligned_address = {addr[63:2], 2'b00}
```

This forces:

```text
addr[1:0] = 00
```


---

# Important Property

Alignment enforcement **never affects sequential execution** if the increment size matches the instruction width.

It mainly protects against **invalid external control transfers**.

---

# Hardware Cost

Alignment enforcement requires only:

```text
bit masking of lower address bits
```

Which is essentially **free hardware**.

For 4-byte alignment:

```text
2 wires forced to zero
```

---

# Summary

Alignment enforcement ensures:

```text
PC always points to a valid instruction boundary
```

Implementation methods:

1. Mask lower bits (hardware alignment)
2. Detect misalignment and raise exception

Most CPUs enforce:

```text
PC % instruction_size == 0
```
