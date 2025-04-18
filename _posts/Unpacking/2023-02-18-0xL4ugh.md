---
title: "Unpacking UPX (MinGW example)"
classes: wide
header:
  teaser: /assets/images/unpacking/MinGW/demo.png
ribbon: DodgerBlue
description: "Simple example on how to modify UPX-packed files"
categories:
  - Unpacking
toc: true
---

# Unpacking


# MinGW

In this project, we'll explore how to modify the strings (text) of executable files. When we run an executable file, strings like "Hello World!" that appear on the GUI are usually located in the `rdata` or `.data` segments. These are parts of the PE file structure used by the program to hold constant values like string literals.

Below is an example of these data segments shown in the program `CFF Explorer`, for my crackme `LSDtrip.exe`:
![Data segments in CFF Explorer showing the .rdata and .data sections](/assets/images/unpacking/MinGW/demo.png)

As you can see, it contains the strings of the program. In these kinds of executables, where the string are statically stored in the binary, we can simply load them up in a
Hex editor like `HxD` and modify these values:

![Modifying strings in a hex editor](/assets/images/unpacking/MinGW/demo2.png)

# The unpacking process

The main problem is that, in most commercial programs, this won't be as straightforward. Programs often don't store the string "as-is" in the binary. Instead, they use a process called "packing", where static variables are encrypted within the binary. At runtime, these encrypted strings are unpacked through a loop of assembly instructions, then loaded into memory and displayed in the GUI. This means that, in the executable binary, instead of seeing `"Hello World"`, you'll see something like `+!��4!��=!��D!��L!��U!��^!��e!��r!��ƒ!`.

To modify these strings, we need to "unpack" the program. This means saving the executable code at a memory state where the program has already decrypted all the static string variables. This way, all the decrypted strings will be contained within our .exe file, and we could then easily change them.

For this process, we will use a debugger called `x32dbg` and its built-in plugin for dumping, `Scylla`.

![Debugger showing pushad instruction](/assets/images/unpacking/MinGW/demo6.png)
We will unpack an example UPX-packed `MinGW` executable. First, we open it in `x32dbg`, and click F9 until we reach the `pushad` instruction. Then, we'll press `Ctrl + F` to search for the assembly instruction `popad` and locate it in the code:

![Searching for popad instruction](/assets/images/unpacking/MinGW/demo7.png)

![Searching for popad instruction](/assets/images/unpacking/MinGW/demo8.png)

![Searching for popad instruction](/assets/images/unpacking/MinGW/demo9.png)

Then, we set a breakpoint at the nearest `jmp` instruction to `popad`. In this case, it's `jmp 0x00401580`. We'll set a breakpoint on this address, and then we'll press `F7` (Step Over).

![Nearest jmp instruction](/assets/images/unpacking/MinGW/demo10.png)

As you can see, the instruction `jmp 0x00401580` takes us to `push ebp`. This is the `OEP (Original Entry Point)` of the program, meaning it's at this instruction where the program finished unpacking itself and is ready for the actual intended code execution after unpacking.
![Setting breakpoint and stepping over](/assets/images/unpacking/MinGW/demo11.png)

Now, we will open the plugin `Scylla` in `x32dbg`. We specify the OEP address, which is the instruction `push ebp`, address `00401580`. Now, we'll click `IAT Autosearch` and `Get Imports`:
![Using Scylla to get imports](/assets/images/unpacking/MinGW/demo5.png)

Next, we click `Dump` to save the unpacked executable:

![Dumping the unpacked executable](/assets/images/unpacking/MinGW/demo14.png)

# Fixing the imports

The file is saved as `dump.exe`. However, when we attempt to open it, an error message appears:

![Error message when opening dump.exe](/assets/images/unpacking/MinGW/demo13.png)

This error occurs because the imports are corrupted. To fix this, we return to Scylla, click `Fix Dump`, and select our dumped file, `dump.exe`:
![Fixing the dumped executable with Scylla](/assets/images/unpacking/MinGW/demo15.png)

After fixing the imports, we have a working unpacked executable that we can modify, `dumped_SCY.exe`

![Successfully running dumped_SCY.exe](/assets/images/unpacking/MinGW/demo16.png)

Now, we can open the unpacked `dumped_SCY.exe` with HxD and modify the strings as needed:

![Modifying strings in HxD](/assets/images/unpacking/MinGW/demo17.png)