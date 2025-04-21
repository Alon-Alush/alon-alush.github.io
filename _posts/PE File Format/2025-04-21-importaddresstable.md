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

# Example

![MessageBoxA implementation](/assets/images/pefileformat/address.png)
`user32.dll` -> `MessageBoxA` -> `75EA05B0`

Here, we can see that function implementation of MessageBoxA resides in the address `0x75EA05B0` in memory.

![MessageBoxA implementation](/assets/images/pefileformat/messagebox.png)

So the question is, how does the program *know* that MessageBoxA resides here `0x75EA05B0`? 

The answer is that the Import Address Table contains **RVAs** from the Image Base to the function's implementation in memory
This address is not hardcoded, but rather dynamically resolved.