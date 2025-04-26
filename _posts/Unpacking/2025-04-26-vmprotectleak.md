---
title: "Breach of VMProtect: the 2 incidents"
classes: wide
header:
  teaser: /assets/images/unpacking/vmprotectleak/dump.png
ribbon: DodgerBlue
description: "Discussing the breaches of VMProtect's source code that emerged in 2023"
categories:
  - Unpacking
toc: true
---

# VMProtect leak

![VMProtect UI](/assets/images/unpacking/vmprotectleak/software.png)

[VMProtect](https://vmpsoft.com/), the commercial virtualization-based protector for executable files, has been leaked — *twice*.

First, a dump of the protector popped up in May 2023:

![VMProtect initial leak](/assets/images/unpacking/vmprotectleak/dump.png)

Then a more complete archive was posted (7 Dec 2023) and spread rapidly across GitHub, forums, and X:

![VMProtect full archive on GIThUB](/assets/images/unpacking/vmprotectleak/full.png)
 
 But *"breached"* ≠ *“instantly broken”*; having the source code doesn't hand attackers a magic "*de-virtualize.exe*. It just drops the difficulty from "dark art" to "grad-school project". 

# The two big leaks (so far)

- **May 2023** : Large chunk of VMProtect 3.x codebase leaked, first public confirmation that the code was in the wild.[[1]](https://forum.tuts4you.com/topic/44205-leaked-vmprotect-sources/?utm_source=chatgpt.com)[[2]](https://www.risky.biz/RBNEWS146/)[[3]](https://www.unknowncheats.me/forum/general-programming-and-reversing/583253-vmprotect-source-leak.html)[[4]](https://x.com/gmhzxy/status/1563608617169096708)