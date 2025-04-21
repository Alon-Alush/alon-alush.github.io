---
title: "PE file format: Import Address Table"
classes: wide
header:
  teaser: /assets/images/injection/codecaves/codecaves.png
ribbon: DodgerBlue
description: "Detailed explanation regarding the Import Address Table (IAT) structure in Portable Executable (PE) file"
categories:
  - PE File Format
toc: true
---

# PE file format: Import Address Table

An `.exe` uses the Import Address Table **to dynamically resolve the addresses where an external function's resides resides in memory**.

The IAT is contained inside the `.idata` section in the PE file:

![MessageBoxA implementation](/assets/images/pefileformat/iat.png)

At runtime, this structure is mapped into memory.

# Example

Here, we can see that function implementation of MessageBoxA resides in the address `0x75EA05B0` in memory.

`user32.dll` -> `MessageBoxA` -> `75EA05B0`

![MessageBoxA implementation](/assets/images/pefileformat/messagebox.png)

**How does the program *know* that MessageBoxA resides here `0x75EA05B0`?**

This is where the Import Address Table comes into play: it contains the **RVAs (Relative Virtual Addresses)** from the Image Base to the function's implementation in memory.

The program uses the Import Address Table to dynamically resolve the addresses of these functions.