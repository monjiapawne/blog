---
title: 'examples'
weight: 2
---

# Examples

## glibc function call fprintf

```gas
# Prints "25" to stdout
# build: gcc -static print_example.s
.globl main
.section .data
fmt_int:
    .ascii "%d\n\0"
.section .text
main:
    movq stdout, %rdi     # arg1: FILE* stdout 
    movq $fmt_int, %rsi   # arg2: format string
    movq $25, %rdx        # arg3: integer to print
    movq $0, %rax         # no floating-point args (required by ABI)
    call fprintf          # fprintf(stdout, "%d\n", 25)

    mov $0, %rax
    ret
```