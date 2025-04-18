---
title: "AV Evasion: Anti-debug tricks"
classes: wide
header:
  teaser: /assets/images/evadingavs/anti-debug/image.jpg
ribbon: DodgerBlue
description: "Learn how malware evades anti-viruses (includes code examples)"
categories:
  - AV-evasion
toc: true
---

# Reducing AV detections

One of the most common techniques malware authors use to create **FUD** (Fully Undetectable) malware is **anti-debugging**. 

Practically *every* anti-virus solution **opens the target executable with *some* form of debugging or inspection enabled**— Sometimes obvious, sometimes very stealthy.

And I haven't heard of any reverse-engineer / malware-analyzer who doesn't have x64dbg or practically any debugging tool lying around 😅

Surprisingly,  believe it or not: it's often the *least* suspicious reactions (like a program immediately exiting without modifying anything), that trigger antivirus detection the most. 

Because of this, **instead of simply terminating when a debugger is detected**, it's usually better to **reroute execution and behave like an entirely *different* (but stil "normal-looking"**) program.

For example: **instead of exiting** the program when a debugger is detected, **route** execution to an entirely different "legit" routine that acts as a random number generator, for example, and *only then* close the program.

A classic "one-liner" anti-debug trick I used in my crackme looks like this:

```c
if (*(BYTE*)(__readgsqword(0x60) + 2)) {
    // Debugger detected
```
The logic behind this line is simple.

```c
__readgsqword(0x60) // <-- gets the Process Environment Block address 
```
```c
+ 2 // <-- 0x02 bytes offset from PEB contains the BeingDebugged field
```
👉 `BeingDebugged` is a boolean (1 byte) that tells whether a debugger is attached to the process!

If `BeingDebugged == 0`, then you are being debugged (Proabably by x64dbg)

If `BeingDebugged == 0`, then no debugger attached.
```c
(*(BYTE*) // A dereference that reads the BeingDebugged boolean
```


