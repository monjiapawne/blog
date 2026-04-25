+++
title = 'concepts'
weight = 2
+++

# x86-64 (AT&T Syntax)

## ABI

ABI - Application Binary Interface is a type of convention for function calls, similar to how there is for syscalls.
The ABI that linux uses is called - **System V ABI**

> **Q:** Why do I care? Why should I follow their convention?  

It allows you to call functions written by other people, even in other programming languages and allows you to estimate the behavior and safety of your registers/stack.

---

## Stack

> **Q:** What is the stack and how does asm use it?

- The stack is an area of memory that is reserved for stacking temporary items. 
- The operating system preallocates space for the stack and then puts the pointer to this memory in the stack pointer , `%rsp`. 
- While pointers generally refer to the beginning of a memory region, at the beginning of a program `%rsp` points to the end of the memory region containing the stack. *(Bartlett, ch. 11.2)*

> **Q:** Why must I keep the stack 16byte aligned before a **function call**?  

This comes from the System V ABI rule, there are instructions that use vectors (a single register that holds values of the same type processed in parallel). These instructions expect 16 contiguous bytes.

> **Q:** What does it mean to align the stack to 16 bytes?

`%rsp` holds the pointer (address) to the top of the stack 

```
rsp = 0x1000   → aligned (0x1000 % 16 == 0)
rsp = 0x1008   → NOT aligned (0x1008 % 16 == 8)
rsp = 0x1010   → aligned
```

---