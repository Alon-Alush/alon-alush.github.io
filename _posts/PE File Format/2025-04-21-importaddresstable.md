---
title: "PE file format: Import Address Table"
classes: wide
header:
  teaser: /assets/images/injection/codecaves/codecaves.png
ribbon: DodgerBlue
description: "Explaining in detail the Import Address Table (IAT) structure in Portable Executable (PE) file"
categories:
  - PE File Format
toc: true
---

# PE file format: Import Address Table

The Import Address Table stores the memory addresses of external functions that the program uses
# Example

Here, we can see that function implementation of MessageBoxA resides in the address `0x75EA05B0` in memory.

`user32.dll` -> `MessageBoxA` -> `75EA05B0`

![MessageBoxA implementation](/assets/images/pefileformat/messagebox.png)

**When calling MessageBoxA, how does the program *know* that MessageBoxA resides here `0x75EA05B0`?**

This is where the Import Address Table comes into play: it contains the **RVAs (Relative Virtual Addresses)** from the Image Base to the function's implementation in memory.

The program uses the Import Address Table to dynamically resolve the addresses of these functions.

Now that we know what the IAT is, let's see how to find it programmatically using C and WinAPI.

# Programatically finding the IAT

Here's how to get to the IAT using WinAPI and C:
```c
// decryptedPE is a BYTE* pointing to the in-memory EXE buffer
IMAGE_DOS_HEADER* dosHeader = (IMAGE_DOS_HEADER*)decryptedPE;
IMAGE_NT_HEADERS64* ntHeaders = (IMAGE_NT_HEADERS64*)(decryptedPE + dosHeader->e_lfanew);
IMAGE_IMPORT_DESCRIPTOR* importDesc = (IMAGE_IMPORT_DESCRIPTOR*)((DWORD_PTR)imageBase + ntHeaders->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress);
IMAGE_THUNK_DATA64* firstThunk = (IMAGE_THUNK_DATA64*)((DWORD_PTR)imageBase + importDesc->FirstThunk); // Locate IAT
```

Here, `firstThunk` is a pointer to the Import Address Table in memory.

# IAT structure

The IAT itself contains several fields:

![MessageBoxA implementation](/assets/images/pefileformat/fields.png)

- `u1.AddressOfData`: **Before loading (on-disk)**, this is a pointer (RVA) to an `IMAGE_IMPORT_BY_NAME` structure (where the function name is stored).
- `u1.Ordinal` : If the function is imported by `ordinal` (number) instead of by name.
- `u1.ForwarderString`: If a function is [*forwarded*](https://devblogs.microsoft.com/oldnewthing/20060719-24/?p=30473), this field contains an RVA to ASCII strings in memory that includes the name of the forwarded module and the function inside it to look for (e.g. `NTDLL.RtlExitUserThread`).
- `u1.Function` After the loader resolves the import, the `u1.Function` field is populated with the **absolute memory address** of the function in memory.