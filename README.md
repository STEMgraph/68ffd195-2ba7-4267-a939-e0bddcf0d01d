<!---
{
  "id": "68ffd195-2ba7-4267-a939-e0bddcf0d01d",
  "teaches": "Understanding the Stack in x86\_64 Assembly",
  "depends_on": ["800c7dd9-5ccf-42c1-b9ea-c2764579cf5d"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-06-12",
  "keywords": ["assembly", "stack", "x86_64", "rsp", "push", "pop"]
}
--->

# Understanding the Stack in x86\_64 Assembly

> In this exercise you will learn how the stack operates in x86\_64 assembly using GAS syntax. Furthermore we will explore how instructions like `push` and `pop` interact with the stack pointer `%rsp` and why the stack is essential for structured programming.

## Introduction

After learning how to use `jmp` and `cmp` to control the flow of execution, a natural next step is organizing code into **reusable blocks**, also known as **functions**. However, for these reusable blocks to work correctly — especially when we want to return to where we left off — we need a reliable way to store temporary information. This is where the **stack** comes into play.

Up until now, we have worked with simple data placed at **predetermined locations in memory**, usually in the `.data` section. These values had fixed labels and were not manipulated in a dynamic way. The stack introduces a new and flexible concept: a **data structure** that allows **dynamic and temporary storage** of values during program execution. Unlike fixed memory regions, the stack is designed to grow and shrink in response to runtime conditions.

The **stack** is a special region of memory used to store **temporary data** such as function arguments, local variables, and return addresses. It operates in a **Last-In, First-Out (LIFO)** manner, meaning the last value pushed onto the stack is the first one to be popped off.

In x86\_64 architecture, the stack grows **downward** in memory: every push decreases the **stack pointer** (`%rsp`), and every pop increases it. The stack pointer is a dedicated register that always points to the **top of the stack**.

From a hardware perspective, the stack is implemented as a reserved area in main memory — that is, **RAM**, not memory physically embedded on the CPU chip. When a program starts, the operating system allocates a section of RAM specifically for use as the stack. The processor’s **control logic** and the `%rsp` register then manage this memory region.

The `%rsp` register is directly manipulated by stack-related instructions (`push`, `pop`, `call`, `ret`). These instructions are **not system calls** — they are native CPU instructions executed entirely in hardware. While the **kernel sets up the initial stack** for each process during its creation, once running, the **CPU itself tracks where each process’s stack is located** using internal registers. That means when `push` or `pop` is executed, the processor already knows — based on the current `%rsp` value — **where to write or read in memory**, without needing to ask the operating system.

When you execute `push`, the hardware control unit performs the following actions:

1. Decrement the `%rsp` register by the size of the data (8 bytes on x86\_64).
2. Store the data at the new address pointed to by `%rsp`.

When you execute `pop`, the reverse happens:

1. Read the value at the address `%rsp`.
2. Copy it into the destination register.
3. Increment `%rsp` by 8 bytes.

Although the stack resides in RAM, modern CPUs optimize stack access using fast cache hierarchies, making these operations very efficient. Internally, the processor may prefetch and buffer accesses to stack addresses, but the logical storage of stack content is in RAM.

This mechanism is heavily used in function calls to temporarily save state and transfer control. Later exercises will build on this by showing how the stack is used to store **return addresses** and **arguments** during a `call`.

### Further Readings and Other Sources

* [https://en.wikibooks.org/wiki/X86\_Assembly/Stack\_Section](https://en.wikibooks.org/wiki/X86_Assembly/Stack_Section)
* [https://cs.lmu.edu/\~ray/notes/stack/](https://cs.lmu.edu/~ray/notes/stack/)
* [YouTube: "Assembly Language Tutorial: Stack and Stack Frames"](https://www.youtube.com/watch?v=HsY2NJrGZ4w)

## Tasks

### 1. Simple Push and Pop

Write a program that pushes an immediate value (e.g., `7`) onto the stack, pops it into a register, and then prints that value using syscall.

```gas
.section .data
msg: .ascii "Result: 7\n"
len = . - msg

.section .text
.globl _start

_start:
    mov $7, %rax
    push %rax        # Store 7 on the stack
    pop %rbx         # Get 7 back into %rbx

    # Print message
    mov $1, %rax
    mov $1, %rdi
    lea msg(%rip), %rsi
    mov $len, %rdx
    syscall

    mov $60, %rax
    xor %rdi, %rdi
    syscall
```

### 2. Stack Pointer Inspection

Extend the program to print the address stored in `%rsp` before and after a `push`. You may move it into `%rdi` and use `syscall` if your setup allows it, or just trace it with `gdb`.

### 3. Stack as Temporary Storage

Push two different values to the stack. Pop them into two different registers and print both. Show that the value popped first is the **last one pushed**.

```gas
    mov $1, %rax
    push %rax        # Push 1
    mov $2, %rax
    push %rax        # Push 2

    pop %rbx         # Gets 2
    pop %rcx         # Gets 1
```

## Questions

1. Why does the stack grow downward in memory?
2. What does `%rsp` point to at any given time?
3. What is the difference between `mov` and `push`?
4. What would happen if you `pop` without a prior `push`?

## Advice

The stack is one of the most important tools in low-level programming. It allows you to store temporary values, manage nested function calls, and implement recursion. Don’t rush it — experiment with different `push`/`pop` combinations and inspect `%rsp` using a debugger. In the next sheet on [function calls](#), you'll see how stack management makes those possible.
