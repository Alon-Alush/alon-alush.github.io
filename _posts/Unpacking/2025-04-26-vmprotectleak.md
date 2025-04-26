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

[VMProtect](https://vmpsoft.com/), the commercial virtualization-based protector for executable files, has been leaked — not once, but *twice*.

First, a dump of the protector popped up in May 2023:

![VMProtect initial leak](/assets/images/unpacking/vmprotectleak/dump.png)

This leak was shared via the Chinese Kanxue Security Forum:

![forum leak](/assets/images/unpacking/vmprotectleak/forum.png)

Then a more complete archive was posted (7 Dec 2023) and spread rapidly across GitHub, forums, and X:

![VMProtect posted on GitHub](/assets/images/unpacking/vmprotectleak/full.png)

# The two big leaks (so far)

- **May 2023** : Large chunk of VMProtect 3.x codebase leaked, first public confirmation that the code was in the wild.[[1]](https://forum.tuts4you.com/topic/44205-leaked-vmprotect-sources/?utm_source=chatgpt.com)[[2]](https://www.risky.biz/RBNEWS146/)[[3]](https://www.unknowncheats.me/forum/general-programming-and-reversing/583253-vmprotect-source-leak.html)[[4]](https://x.com/gmhzxy/status/1563608617169096708)

- **December 2023** : **A second, packaged dump advertised as the "full" source was released on Github**. Still missing a few files such as the ARM back-end, demanglers, but enough to compile parts of the tool[[1]](https://github.com/jmpoep/vmprotect-3.5.1)

![VMProtect full archive on github](/assets/images/unpacking/vmprotectleak/github.png)

There was no evidence of an *internal* hack at VMPSoft, the consensus is an outsider stole the code (likely from a customer portal or reseller) and let it roam free. 

# Is VMProtect "cracked" now?

Yes and no; having source code made it possible to partially compile VMProtect:

![VMProtect full archive on github](/assets/images/unpacking/vmprotectleak/partial.png)

**There's still no "one-click unpack" tool** available because the leaks lack some core virtualization/processor modules.