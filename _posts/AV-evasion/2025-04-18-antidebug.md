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
Here's an example of function hooking in VirusTotal Jujubox:

![VirusTotal Jujubox function hooking](/assets/images/evadingavs/anti-debug/jujubox.png)

# What you have to know

Surprisingly: it's often the *least* suspicious reactions (like a program immediately exiting without modifying anything), that trigger antivirus detection the most. 

This is why **instead of simply terminating when a debugger is detected**, it's better to **reroute execution and behave like an entirely *different* (but still "normal-looking"**) program.

For example: **instead of exiting** the program when a debugger is detected:
```c
if (IsDebuggerPresent()) {
    // Debugger found
    ExitProcess();
    return 1;
}
```
Do something like:
```c
if (IsDebuggerPresent()) {
    // Debugger found
    printf("%s", "Welcome to my Fibonacci number printer!");
    unsigned long long sum = 0;
    unsigned long long num1 = 0;
    unsigned long long num2 = 1;
    unsigned long long upto;
    printf("Generate fibonacci numbers up to:");
    scanf("%d", upto);
    printf("%d\n", num1);
    printf("%d\n", num2);
    while (sum < upto) {
        sum = num1 + num2;
        printf("%d\n", sum);
        num1 = num2;
        num2 = sum;
    }
    return 1;
}
// Nefarious code goes here!
```

 **route** execution to an entirely different "legit" routine that acts as a random number generator, for example, and *only then* close the program.

# Anti-debugging methods

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

If `BeingDebugged == 1`, then you are being debugged (proabably by x64dbg)

If `BeingDebugged == 0`, then no debugger attached.
```c
(*(BYTE*) // A dereference that reads the BeingDebugged boolean
```


