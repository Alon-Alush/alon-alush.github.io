---
title: "PE file format: DOS Header"
classes: wide
header:
  teaser: /assets/images/pefileformat/iat/image.png
ribbon: DodgerBlue
description: "Explaining the DOS header in Portable Executable (PE) files."
categories:
  - PE File Format
toc: true
---

# DOS header

**First of all, what's the DOS header?**

The DOS header is a small structure at very beginning of executable files in the DOS MZ format (like `.exe` files), and preserved on PE files for backwards compatibility. It's often called the **MZ header** because the first two bytes are the ASCII characters "`MZ`"—the initials of Mark Zbikowski, one of its designers.

![MZ](/assets/images/pefileformat/dosheader/image.png)

If you try to run the executable in a DOS environment, the stub typically displays this famous message:
```
"This program cannot be run in DOS mode."
```
![DOS header stub](/assets/images/pefileformat/dosheader/image-1.png)

# Key fields in the DOS header

`e_magic`: a 2-byte signature that must equal to `MZ` (`0x4D5A` in hexadecimal). This sequence lets the loader know that the file is a valid DOS executable (and by extension, a valid PE file).

`e_lfanew`: a 4-byte field that contains the offset (relative to the start of `MZ` signature) where the NT header begins.

![e_lfanew field](/assets/images/pefileformat/dosheader/image-2.png)

Here, we can see that `e_lfanew` field corresponds to the byte sequence `C8 00 00 00`. That means the offset to the NT headers is `0xC8`, ignoring the null values.

So, **to find the NT header in HxD**, let's go to offset `C8` that we got from the `e_lfanew` field, this offset is relative to the file start (`begin`):

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

The struct has 14 `WORD` fields (2 bytes each) and 10 `DWORD` fields (4 bytes each), plus the 2-byte `e_magic`.

Total size: 2 + (13 × 2) + (10 × 4) = 2 + 26 + 40 = exactly **64 bytes without paddling**.

# Why is the DOS header still there?

**A lot of you right now may be confused:** Why does every PE file need a DOS header, despite modern Windows system not even running DOS programs?

2 words: **backwards compatibility**. 

See, the PE format was designed *as an extension* of the old [DOS MZ executable format](https://en.wikipedia.org/wiki/DOS_MZ_executable). Back then, the Portable Executable format was a *new kid on the block*. Both the old DOS MZ format and PE format ended with a `.exe` extension, so whenever a user attempted to run a PE `.exe` on MS-DOS, **the DOS loaders would freak out** because they expected a DOS MZ format for `.exe` extensions, not PE. 

So, in order to not make the DOS loaders freak out when they see a PE executable, they required *every* PE file contain this DOS stub inside the DOS header:

```c
This program cannot be run in DOS mode
```

So that if a DOS loader (or a user in DOS) tries to execute a PE file, they don't crash or hang; instead, they immediately see this message in the console:

![This program cannot be run in DOS mode](/assets/images/pefileformat/dosheader/loader.png)

Looking back, this *"might've"* been a mistake move by Microsoft😅. The PE format evolved to be a much more powerful and modular format afterward, *way* more than originally speculated, and leaving the old DOS-MZ format to be forgotten in the shadows. Yet, every PE file in existence *still* contains those 36 bytes worth of ASCII bloat, to remind *DOS loaders* from 40 years ago that our modern `.exe` file cannot be run on DOS systems.

# DOS stub code injection

The DOS header (and the DOS stub) **can technically be modified to inject custom code**. Back in the days of DOS-MZ, malware authors would inject their custom 16-bit DOS code into the DOS header itself. Then, when a DOS machine tries to run the modified `.exe`, it will execute the injected DOS code instead of the boring "This program cannot be run in DOS mode" message. 