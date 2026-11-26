# Module 1: Memory Addressing & Data Representation

> Understanding how data lives in memory, how to access it, and how to manipulate it using x86-64 Assembly.

## 1. Basic NASM Program Structure

Every basic Linux Assembly program needs a starting point.

```asm
section .data

section .bss

section .text
global _start

_start:

    ; Instructions go here

    ; Exit program
    mov rax, 60
    mov rdi, 0
    syscall
```

**Three important sections:**

| Section | Purpose |
|---|---|
| `.data` | Stores initialized data |
| `.bss` | Reserves memory without providing initial values |
| `.text` | Contains executable instructions |

## 2. Data Declaration: DB, DW, DD, DQ

NASM provides different declarations depending on how much memory a value needs.

| Declaration | Full name | Size |
|---|---|---:|
| `DB` | Define Byte | 1 byte |
| `DW` | Define Word | 2 bytes |
| `DD` | Define Doubleword | 4 bytes |
| `DQ` | Define Quadword | 8 bytes |

**Example:**

```asm
section .data

a db 10
b dw 500
c dd 70000
d dq 5000000000
```

Here:

- `a` occupies 1 byte.
- `b` occupies 2 bytes.
- `c` occupies 4 bytes.
- `d` occupies 8 bytes.

Total memory used: 15 bytes.

**Remember:** The declaration determines how much memory the variable occupies. It does not determine which register you must use.

## 3. Reserving Memory: RESB, RESW, RESD, RESQ

Sometimes we don't know the data beforehand. We just need some empty storage.

That's where the `RES` declarations come in.

| Declaration | Memory reserved |
|---|---:|
| `RESB 10` | 10 bytes |
| `RESW 10` | 20 bytes |
| `RESD 10` | 40 bytes |
| `RESQ 10` | 80 bytes |

**Example:**

```asm
section .bss

buffer resb 50
numbers resq 5
```

- `buffer` reserves 50 bytes.
- `numbers` reserves 40 bytes.

**DB vs RESB:**

```asm
message db "Adi"
buffer resb 20
```

`message` stores three initialized characters.

`buffer` reserves 20 bytes without defining their initial contents.

## 4. Address vs Value

This is one of the most important concepts in Assembly.

Every location in memory has an address, and data is stored at that address.

Suppose:

```asm
section .data

number dq 500
```

Imagine:

```text
Address:  0x402000
Value:    500
```

There are two different things:

- Address = `0x402000`
- Value = `500`

We must know which one we need.

## 5. MOV vs LEA

These instructions look similar in some situations, but they perform different operations.

**MOV: Retrieve the value stored in memory.**

```asm
mov rax, [number]
```

If `number` contains 500:

```text
RAX = 500
```

**LEA: Calculate the effective address.**

```asm
lea rbx, [number]
```

If `number` is located at `0x402000`:

```text
RBX = 0x402000
```

It does not retrieve the value 500.

**Quick comparison:**

| Instruction | What it obtains |
|---|---|
| `mov rax, [number]` | Value stored at number |
| `lea rax, [number]` | Address of number |

A useful way to remember:

- `MOV` with memory brackets retrieves data.
- `LEA` calculates an address.

## 6. Direct and Indirect Memory Access

**Direct memory access:**

Access a variable using its label.

```asm
section .data
number dq 500

section .text
_start:
    mov rax, [number]
```

Here, we directly reference the memory location identified by `number`.

**Indirect memory access:**

Use a register that contains a memory address.

```asm
lea rbx, [number]
mov rax, [rbx]
```

First, RBX receives the address of `number`.

Then, MOV retrieves the value stored at that address.

This is the basic idea behind pointers.

## 7. Pointers and Pointer Arithmetic

A pointer is simply a value that stores a memory address.

```asm
section .data
numbers dq 11, 22, 33, 44, 55

section .text
_start:
    lea rbx, [numbers]

    mov rax, [rbx]
    mov rcx, [rbx + 8]
```

Assuming each element is a QWORD:

| Address offset | Value |
|---:|---:|
| +0 | 11 |
| +8 | 22 |
| +16 | 33 |
| +24 | 44 |
| +32 | 55 |

Why is the second element at `+8`?

Because each QWORD occupies 8 bytes.

**Important:** Pointer offsets are measured in bytes, not in the number of elements.

For QWORD arrays:

- First element: `+0`
- Second element: `+8`
- Third element: `+16`
- Fourth element: `+24`

## 8. Memory Size Qualifiers

Sometimes NASM cannot determine how much memory an instruction should access.

For example:

```asm
mov [buffer], 10
```

This is ambiguous. NASM doesn't know whether we intend to write 1, 2, 4 or 8 bytes.

We can specify the size:

```asm
mov byte [buffer], 10
mov word [buffer], 500
mov dword [buffer], 70000
mov qword [buffer], 5000000000
```

**Size qualifiers:**

| Qualifier | Size |
|---|---:|
| `BYTE` | 1 byte |
| `WORD` | 2 bytes |
| `DWORD` | 4 bytes |
| `QWORD` | 8 bytes |

When a register is involved, NASM can often infer the memory size from the register:

```asm
mov rax, 500
mov [buffer], rax
```

Here, RAX is 64-bit, so NASM knows the operation uses 8 bytes.

**Important limitation:** x86-64 does not support an ordinary `MOV` that directly writes an arbitrary 64-bit immediate value into memory. Load the immediate into a register first.

```asm
mov rax, 5000000000
mov qword [buffer], rax
```

## 9. Little Endian Representation

Little Endian describes how multi-byte values are stored in memory.

The least significant byte is stored at the lowest memory address.

Example:

```asm
section .data
number dq 0x1122334455667788
```

Memory representation:

| Address offset | Stored byte |
|---:|---|
| +0 | 88 |
| +1 | 77 |
| +2 | 66 |
| +3 | 55 |
| +4 | 44 |
| +5 | 33 |
| +6 | 22 |
| +7 | 11 |

Notice that the byte order in memory is reversed compared with how we normally write the hexadecimal number.

**Remember:** Little Endian changes byte order in memory. It doesn't change the actual numerical value held by the register.

## 10. Memory Aliases Using %DEFINE

NASM provides `%define` to create text-substitution aliases.

Example:

```asm
section .bss
buffer resb 15

%define flag buffer
%define code buffer+1
%define count buffer+3
%define total buffer+7
```

Now we can reference different locations using meaningful names:

```asm
mov byte [flag], 7
mov word [code], 500
mov dword [count], 70000
mov rax, 5000000000
mov qword [total], rax
```

Memory layout:

| Variable | Offset | Size |
|---|---:|---:|
| flag | 0 | 1 byte |
| code | 1 | 2 bytes |
| count | 3 | 4 bytes |
| total | 7 | 8 bytes |

Total: 15 bytes.

This is useful when working with manually arranged data structures.

**Important:** `%define` does not allocate memory. It simply substitutes one name with another expression.

## 11. GDB: Inspecting Memory and Registers

GDB is extremely useful when learning Assembly because it allows us to inspect what is actually happening.

Some commands worth remembering:

| GDB command | Purpose |
|---|---|
| `info registers` | Display registers |
| `info registers rax rbx` | Inspect specific registers |
| `x/5gx address` | Examine 5 eight-byte values |
| `x/15xb address` | Examine 15 bytes in hexadecimal |
| `x/10cb address` | Examine 10 bytes as characters |
| `break _start` | Set a breakpoint |
| `run` | Execute the program |
| `si` | Execute one instruction |
| `ni` | Execute one instruction without stepping into a call |
| `disassemble _start` | Display Assembly instructions |

Example:

```gdb
break _start
run
info registers rax rbx
x/8xb &buffer
```

**Why use GDB?**

Don't just assume that your instructions worked. Inspect the registers and memory to verify the actual results.

---

# Important Things to Remember

These are the concepts worth revising regularly:

1. **Address vs Value:** Knowing where data is stored is different from knowing what is stored there.
2. **MOV vs LEA:** MOV retrieves data; LEA calculates an address.
3. **Pointers:** A pointer stores a memory address.
4. **Memory offsets:** Offsets are measured in bytes.
5. **Data sizes:** DB = 1, DW = 2, DD = 4, DQ = 8.
6. **RES declarations:** Reserve memory without defining its initial contents.
7. **Size qualifiers:** Specify BYTE, WORD, DWORD or QWORD when the assembler cannot infer the size.
8. **Little Endian:** The least significant byte occupies the lowest memory address.
9. **Register management:** Keep pointers and retrieved data in separate registers when necessary.
10. **GDB:** Verify your assumptions by inspecting actual memory and registers.

## Common Mistakes I Encountered

| Mistake | What to remember |
|---|---|
| Using MOV when I needed an address | Use LEA |
| Overwriting a pointer register with retrieved data | Keep the pointer in a separate register |
| Confusing bytes with QWORD elements | One QWORD = 8 bytes |
| Accessing memory beyond the reserved region | Calculate offsets and memory boundaries |
| Using an immediate value without a size qualifier | Specify the memory size |
| Trying to use an unsupported 64-bit immediate-to-memory MOV | Load the value into a register first |
| Assuming exit status can represent any full integer | Linux process exit status is limited to 8 observable bits |

---

**Module 1 Status: COMPLETED**

Core skills covered:
- Memory representation
- Data declarations
- Addressing modes
- Pointers
- Memory manipulation
- GDB memory inspection

**Practical takeaway:** I can declare memory, access its contents, work with addresses, manipulate values using pointers, and inspect the results using GDB.

Next module: Linux System Calls & I/O.
