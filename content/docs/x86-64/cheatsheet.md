+++
date = '2026-04-22T21:19:13-04:00'
title = 'cheatsheet'
weight = 2
+++

# x86-64 (AT&T Syntax)

---

## Registers

### General Purpose

| Register      | 64           | 32     | 16     | 8      |
| ------------- | ------------ | ------ | ------ | ------ |
| accumulator   | `%rax`       | `%eax` | `%ax`  | `%al`  |
| base          | `%rbx`       | `%ebx` | `%bx`  | `%bl`  |
| counter       | `%rcx`       | `%ecx` | `%cx`  | `%cl`  |
| data          | `%rdx`       | `%edx` | `%dx`  | `%dl`  |
| source index  | `%rsi`       | `%esi` | `%si`  | `%sil` |
| dest index    | `%rdi`       | `%edi` | `%di`  | `%dil` |
| stack pointer | `%rsp`       | —      | —      | —      |
| base pointer  | `%rbp`       | —      | —      | —      |
| extended      | `%r8`–`%r15` | `%r8d` | `%r8w` | `%r8b` |

### Size Suffixes (GNU)

- `q` — 8 bytes
- `l` — 4 bytes
- `w` — 2 bytes
- `b` — 1 byte

---

### Caller-Saved

registers the caller must save if needed after a call

```c
%rax %rcx %rdx %rsi %rdi %r8 %r9 %r10 %r11
```

### Callee-Saved

registers the callee must save and restore if used

```c
%rbx %rbp %r12 %r13 %r14 %r15
```

### Function Parameters

```gas
movq $1, %rdi
movq $2, %rsi
movq $3, %rdx
movq $4, %rcx
movq $5, %r8
movq $6, %r9
pushq $10     # after the first 6 registers use the stack
pushq $9
pushq $8
pushq $7
call myfunc
```

### Return Values

```gas
%rax   # primary
%rdx   # secondary (rare)
```

---

## Syscalls

```gas
%rax     # syscall number
%rdi     # param 1
%rsi     # param 2
%rdx     # param 3
%r10     # param 4
%r8      # param 5
%r9      # param 6
syscall
```

---

## Stack

### Stack Frame

start of a function:

```gas
pushq %rbp          # save the pointer to the previous stack frame
movq  %rsp, %rbp    # copy the stack pointer to the base pointer for a fixed reference point
subq  $16, %rsp     # reserve memory on the stack (16-byte aligned)

# or
enter $16, $0       # second arg always 0 (enter is slower than manual)
```

end of a function:

```gas
movq %rbp, %rsp     # restore the stack pointer
popq %rbp           # restore the base pointer
ret

# or
leave               # (leave is faster than manual)
ret
```

{{< notice tip >}}
- If all work fits in registers and no callee-saved regs touched you can skip frame setup entirely  
- If you don't use `enter` don't use `leave` it will result in a segment fault
{{< /notice >}}

### Writing/Reading to the Stack

#### Via Base Pointer (`%rbp`)

```gas
.equ LOCAL_FOO, -8
.equ LOCAL_BAR, -16

enter $16, $0

movq %rax, LOCAL_FOO(%rbp)   # write
movq LOCAL_FOO(%rbp), %rax   # read
```

#### Via `push` / `pop`

```gas
pushq %rax    # push register onto stack
popq  %rbx    # pop top of stack into register
```

When you push something onto the stack, it does two things:

1. decrements `%rsp` to point to the next location on the stack
2. copies the value to the location specified by `%rsp`

`pop` instructions do the inverse

---

## Functions

### `call` / `ret`

```gas
call myfunc   # pushes return address onto stack, jumps to myfunc
ret           # pops return address, jumps to it
```

## Instructions

### Comparison, Branching, and Looping Instructions

| Instruction | Desc                                                                        |
| ----------- | --------------------------------------------------------------------------- |
| `cmp s, d`  | cmp `src` to `dst` perform virtual subtraction (src - dst), only sets flags |
| `test s, d` | `AND` the `src` & `dst` and sets flags                                      |
| `loop d`    | decrements `%rcx` and jumps to `dst` if %rcx is not 0                       |
| `loope d`   | same as `loop`, but also will not jump if the zero flag is set              |
| `loope d`   | same as `loop`, but also will not jump if the zero flag is **not** set      |

