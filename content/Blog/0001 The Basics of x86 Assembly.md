---
title: "The Basics of Intel x86 Assembly"
created: 2023-11-20
modified: 2023-11-20
tags: ["CPP", "Assembly"]
draft: false
---

## Introduction

I'm developing an product which requires me to have fairly good understanding about x86 assembly. While setting up my [Netwide Assembler](https://en.wikipedia.org/wiki/Netwide_Assembler) on Windows I encountered numerous challenges since there were not many resources for setting it up on Windows. What is Netwide Assembler? It's an assembler and disassembler for Intel x86 architecture which can be used for writing assembly code.

In his blog post I will go through the process of setting up Netwide Assembler on Windows and provide informations about x86 assembly to help you get started with programming in x86 assembly.

## Installing NASM

1. Install Netwide Assembler from the following link: [http://nasm.us](https://www.nasm.us)
2. Open `System Properties`
3. Select `Environment Variables`
4. Select `Path` and click on `Edit`
5. Add `C:\Users\[USERNAME]\AppData\Local\bin\NASM`
6. `Save` and `Exit`

## Installing MSYS2

1. Install MSYS2 from the following link: [https://msys2.org](https://www.msys2.org/)
2. Launch `MSYS2`
3. Execute the following command: `pacman -S mingw-w64-x86_64-toolchain`
4. Open `System Properties`
5. Select `Environment Variables`
6. Select `Path` and click on `Edit`
7. Add `C:\msys64\mingw32\bin`
8. `Save` and `Exit`

## Compile and Execute

Now as Netwide Assembler and MSYS2 has been installed and properly configured we can compile and execute x86 assembly code.

```asm title="x86 Assembly"
global  _main
extern  _printf

section .data
    message db 'Hello World', 0xA, 0

section .text
    push    ebp
    mov     esp, ebp

    push    message
    call    _printf
    add     esp, 4

    xor     eax, eax
    ret
```

```cmd title="Terminal"
PS C:\Users\husenjan > nasm main.asm -f win32 -o main.o
PS C:\Users\husenjan > gcc main.obj -o main.exe
```

The `nasm` command translates the x86 assembly code onto a object file and the `gcc` command compiles the object file to an executable program. 

## Basics of Assembly x86

The `extern` instruction in x86 assembly is used to import C++ functions into x86 assembly environment.

```asm
global  _main
extern  _printf

; Used for storing strings and variables
section .data
    output      db      'Hello World', 0

section .text
_main:
    ; Creating stack
    push    ebp
    mov     esp, ebp

    ; Calling printf
    push    output
    call    _printf
    add     esp, 4

    ; Exiting program
    xor     eax, eax
    ret
```

The `extern _printf` is importing in `printf()` function in C++ and the `call _printf` is calling the `printf()` function. When calling a function in x86 assembly it's critical to ensure the stack is being handled correctly before and after the function call otherwise it can create problems.

While-loops can become necessary for our application incase certain actions needs to be executed multiple of times. In x86 assembly the while loop can be created using the stack, jump instruction, and comparison instruction.

```asm 
global  _main
extern  _printf

; Used for storing strings and variables
section .data
    output      db      'Hello World', 0xA, 0

section .text
_main:
    ; Creating stack
    push    ebp
    mov     esp, ebp

    mov     eax, 0
    ; Creating while-loop
_while_loop:
    push    eax

    ; Calling printf
    push    output
    call    _printf
    add     esp, 4

    ; Incrementing eax
    pop     eax
    inc     eax

    ; Exiting if EAX equals 10
    cmp     eax, 10
    jl      _while_loop

    ; Exiting program
    xor     eax, eax
    ret
```

In `push eax` the EAX register is pushed onto the stack since once `call _printf` is executed the EAX register will be modified so once the function is called the `pop eax` is executed to restore the original value of EAX register and the value is incremented by one at `inc eax` and aslong the EAX register is less than 10 the `_while_loop` will continously be executed.

While-loops are great but for-loops are even better because it's much cleaner and it can easily be created using the stack and loop instruction. The loop instruction is essential since it will continue the loop to ECX register becomes null.

```asm
global  _main
extern  _printf

; Used for storing strings and variables
section .data
    output      db      'Hello World', 0xA, 0

section .text
_main:
    ; Creating stack
    push    ebp
    mov     esp, ebp

    ; Assigning ECX the value 10
    mov     ecx, 10

    ; Creating for-loop
_for_loop:
    push    ecx

    ; Calling printf
    push    output
    call    _printf
    add     esp, 4

    ; Continues to jump to _for_loop if ECX is greater than 0
    pop     ecx
    loop    _for_loop

    ; Exiting program
    xor     eax, eax
    ret
```

The `mov ecx, 10` is assigning the value 10 to ECX register which means it will loop 10 times through `_for_loop` and each time the `loop _for_loop` is executed the ECX is decremented by 1 and this will continue to ECX register becomes null.

Now as we have familiarized ourselves with while-loops and for-loops it's time to start working with arrays. In x86 assembly a array can be created in `.data` section and a while-loop or for-loop can be used to work with the array.

```asm
global  _main
extern  _printf

; Used for storing strings and variables
section .data
    array           db      10  dup(0)
    formatout       db      '%d', 0xA, 0

section .text
_main:
    ; Creating stack
    push    ebp
    mov     ebp, esp
    
    mov     edi, 0
    mov     ecx, 10
    mov     ebx, 1

    ; Looping through array
_loop:
    push    ecx
    ; EAX = [array + edi]
    mov     eax, [array + edi]
    mov     eax, ebx
    add     ebx, 1
    push    ebx
    
    ; Calling _printf
    push    eax
    push    formatout
    call    _printf
    add     esp, 8

    ; Restoring original values of registers
    pop     ebx
    pop     ecx

    ; Preparing to access next array
    add     edi, 2

    ; Jumping to _loop if ECX is greater than 0
    loop    _loop

    xor     eax, eax
    ret
```

The `array db 10 dup(0)` creates the array and the array can be accessed directly using `mov eax, [array_name + edi]` and once EDI register is incremented by 2 the next array is accessible.

## What next?

Programming in x86 assembly can be difficult and challenging in the beginning. However, the more practise that is done the easier it becomes therefore it's highly recommended to start off slow such as programming a calculator using what has been taught in this blog post. If you don't understand something than I recommend googling the question and researching it to you find the answer since that is the only way to learn difficult things. 

Programming a calculator in x86 might seem simple but it's extremely difficult. Here is an simple calculator which I worte in x86 assembly:

```cpp showLineNumbers
global  _main
extern  _printf
extern  _scanf
extern  _system
extern  _sleep

section .data
    cleaner         db          'cls', 0
    menu_01         db          '-- CALCULATOR --', 0xA, 0
    menu_02         db          '1. Addition', 0xA, 0
    menu_03         db          '2. Substraction', 0xA, 0
    menu_04         db          '3. Multiply', 0xA, 0
    menu_05         db          'Select one: ', 0
    menu_06         db          'Enter an number: ', 0
    menu_07         db          'Enter another number: ', 0
    menu_08         db          '%d + %d = %d', 0xA, 0
    menu_09         db          '%d - %d = %d', 0xA, 0
    menu_10         db          '%d * %d = %d', 0xA, 0
    integer_01      dd          0
    integer_02      dd          0
    integer_03      dd          0
    formatin_01     db          '%d', 0

section .text
_main:
    push    ebp
    mov     esp, ebp
    
    push    cleaner
    call    _system
    add     esp, 4

    push    menu_01
    call    _printf
    add     esp, 4

    push    menu_02
    call    _printf
    add     esp, 4

    push    menu_03
    call    _printf
    add     esp, 4

    push    menu_04
    call    _printf
    add     esp, 4

    push    menu_05
    call    _printf
    add     esp, 4

    push    integer_01
    push    formatin_01
    call    _scanf
    add     esp, 8

    push    menu_06
    call    _printf
    add     esp, 4

    push    integer_02
    push    formatin_01
    call    _scanf
    add     esp, 8

    push    menu_07
    call    _printf
    add     esp, 4

    push    integer_03
    push    formatin_01
    call    _scanf
    add     esp, 8

    mov     eax, dword[integer_01]

    cmp     eax, 1
    je      _addition

    cmp     eax, 2
    je      _subscration
    
    cmp     eax, 3
    je      _multiply

_addition:
    mov     ebx, dword[integer_02]
    mov     edx, dword[integer_03]

    mov     eax, ebx
    add     eax, edx

    push    eax
    push    edx
    push    ebx
    push    menu_08
    call    _printf
    add     esp, 16

    jmp     _exit

_subscration:
    mov     ebx, dword[integer_02]
    mov     edx, dword[integer_03]

    mov     eax, ebx
    sub     eax, edx

    push    eax
    push    edx
    push    ebx
    push    menu_09
    call    _printf
    add     esp, 16

    jmp     _exit

_multiply:
    mov     ebx, dword[integer_02]
    mov     edx, dword[integer_03]

    mov     eax, dword[integer_02]
    mul     eax

    mov     edx, dword[integer_03]

    push    eax
    push    edx
    push    ebx
    push    menu_10
    call    _printf
    add     esp, 16

    jmp     _exit

_exit:
    push    3
    call    _sleep
    add     esp, 4

    jmp     _main

    ret
```

Hopefully this blog post has been helpful teaching you about x86 assembly. I'll try to update the blog post more once I'm more familiar with x86 assembly.
