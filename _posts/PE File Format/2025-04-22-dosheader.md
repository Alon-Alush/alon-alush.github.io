---
title: "PE file format: DOS Header"
classes: wide
header:
  teaser: /assets/images/pefileformat/iat/image.png
ribbon: DodgerBlue
description: "Explaining the DOS header in Windows executable formats"
categories:
  - PE File Format
toc: true
---

# DOS header

**First of all, what's the DOS header?**

The DOS header is a structure right at the start of DOS-MZ executables, and preserved on the newer PE format for backwards compatibility. The beginining of the DOS header is marked by the ASCII sequence "`MZ`" (`4D 5A` in hex). Those are the initials of Mark Zbikowski, the designer of the DOS-MZ format.

![MZ](/assets/images/pefileformat/dosheader/image.png)

In Portable Executable files, the DOS header contains an additional DOS stub containing the following ASCII sequence:

```
"This program cannot be run in DOS mode."
```
![DOS header stub](/assets/images/pefileformat/dosheader/image-1.png)


# Key fields in the DOS header

`e_magic`: a 2-byte signature that must equal to `MZ` (`0x4D5A` in hexadecimal). This sequence lets the loader know that the file is a valid DOS executable (and by extension, a valid PE file).

`e_lfanew`: a 4-byte field that contains the offset (relative to the start of `MZ` signature) where the NT header begins. It's located of 

![e_lfanew field](/assets/images/pefileformat/dosheader/image-2.png)

In this example, we can see that `e_lfanew` field corresponds to the byte sequence `C8 00 00 00`. That means that `0xC8` (ignoring the null values) is the offset to the NT header in modern PE files.

**To find the NT header in HxD**, let's go to offset `C8` that we got from the `e_lfanew` field, this offset is relative to the file start (`0x0`):

![Going to offset 0xC8](/assets/images/pefileformat/dosheader/image-3.png)

![New PE header](/assets/images/pefileformat/dosheader/header.gif)

Can see you see the `PE` signature that I highlighted? it marks the start of the **IMAGE_NT_HEADERS** structure, and Windows uses `e_lfanew` to find the offset to that structure.

Below is a diagram I made, illustrating it:

![Sections diagram I made](/assets/images/pefileformat/dosheader/diagram.png)

# C representation of IMAGE_DOS_HEADER

We can take a look at the contents of that structure by looking at the `IMAGE_DOS_HEADER` structure definition from `winnt.h`
```c
typedef struct _IMAGE_DOS_HEADER {  
    WORD e_magic;    // Magic number (MZ)
    WORD e_cblp;     // Bytes on last page of file
    WORD e_cp;       // Pages in file
    WORD e_crlc;     // Relocations
    WORD e_cparhdr;  // Size of header in paragraphs
    WORD e_minalloc; // Minimum extra paragraphs needed
    WORD e_maxalloc; // Maximum extra paragraphs needed
    WORD e_ss;       // Initial (relative) SS value
    WORD e_sp;       // Initial SP value
    WORD e_csum;     // Checksum
    WORD e_ip;       // Initial IP value
    WORD e_cs;       // Initial (relative) CS value
    WORD e_lfarlc;   // File address of relocation table
    WORD e_ovno;     // Overlay number
    WORD e_res[4];   // Reserved words
    WORD e_oemid;    // OEM identifier (for e_oeminfo)
    WORD e_oeminfo;  // OEM information; e_oemid specific
    WORD e_res2[10]; // Reserved words
    LONG e_lfanew;   // File address of new exe header (important!)
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

The struct has 30 `WORD` fields (2 bytes each), plus the 4-byte `e_lfanew` (of size `LONG`).

Total size: 30 * 2 + 4 = exactly **64 bytes**.

# Historical background

**A lot of you right now may be confused:** Why does every PE file need a DOS header, despite modern Windows system not even running DOS programs?

2 words: **backwards compatibility**. 

See, the PE format was designed *as an extension* of the old [DOS MZ executable format](https://en.wikipedia.org/wiki/DOS_MZ_executable). Back then, the Portable Executable format was a *new kid on the block*. Both DOS-MZ and PE executables were distributed with a `.exe` extension, so whenever a user attempted to run a PE `.exe` on MS-DOS, **the DOS loaders would behave unexpectedly** because they expected a DOS MZ format for `.exe` extensions, not Portable Executable. 

So, in order to make the DOS loaders predictably exit the program upon running a PE executable, Microsoft distributed *every* PE file with this DOS stub inside the DOS header:

```c
This program cannot be run in DOS mode
```

So that if a DOS loader tries to execute a PE file, they don't crash or hang; instead, they immediately print this message in the console and exit the process.

![This program cannot be run in DOS mode](/assets/images/pefileformat/dosheader/loader.png)

Looking back, making this ASCII sequence native to *every* PE file *"might've"* been a mistake move by Microsoft😅. The PE format evolved to be a much more powerful and modular format afterward, *way* more than originally speculated, leaving the DOS-MZ forma to die in the shadows. Yet, every PE file in existence *still* contains those 36 bytes worth of ASCII bloat, to remind *DOS loaders* from 40 years ago that our modern `.exe` file cannot be run on DOS systems.

# DOS stub code injection

The DOS header (and the DOS stub) **can be modified to inject custom code**. Back in the days of DOS-MZ, malware authors would inject their custom 16-bit DOS code into the DOS header itself. Then, when a DOS machine tries to run the modified `.exe`, it will execute the injected DOS code instead of the boring "This program cannot be run in DOS mode" message. 

To demonstrate how this is done, I'll take a random PE executable and replace its DOS stub with a custom 16-bit shellcode to it that prints "`@AlonAlush`" 5 times in green.

To correctly exit the program after executing our payload, we'll add the bytes `0x4C01` right after the `PE\0\0` signature. In 16-bit MS-DOS assembly, the opcode `4C` corresponds to the `INT 21h` function `4CH`, which terminates a process with a return code. Following it, `01` is the exit code passed.

![DOS stub shellcode injection](/assets/images/pefileformat/dosheader/doshellcode.png)

Now let's run this patched PE in `DOSBox`, a very popular MS-DOS emulator:

![Injected code in DOSBox](/assets/images/pefileformat/dosheader/result.png)

As you can see, instead of printing the generic "This program cannot be run in DOS mode", our exe ran the injected code that printed `@AlonAlush` name in green 5 times, as we expected. Of course, you can modify the DOS stub to run any 16-bit machine code you'd like.
