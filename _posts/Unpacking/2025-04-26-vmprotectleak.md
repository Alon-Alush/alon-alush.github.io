---
title: "The breach of VMProtect: 2 major incidents"
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

In 2023, [VMProtect](https://vmpsoft.com/), a commercial virtualization-based protector for executable files, was leaked — not once, but *twice*.

First, a dump of the protector popped up in May 2023:

![VMProtect initial leak](/assets/images/unpacking/vmprotectleak/dump.png)

This leak was shared via the Chinese Kanxue Security Forum:

![forum leak](/assets/images/unpacking/vmprotectleak/forum.png)

Then, about half a year later, the more complete archive was leaked (7 Dec 2023) and posted on GitHub:

![VMProtect posted on GitHub](/assets/images/unpacking/vmprotectleak/full.png)



# Timeline

- **May 2023** : Large chunk of VMProtect 3.x codebase leaked, first public confirmation that the code was in the wild.[[1]](https://forum.tuts4you.com/topic/44205-leaked-vmprotect-sources/?utm_source=chatgpt.com)[[2]](https://www.risky.biz/RBNEWS146/)[[3]](https://www.unknowncheats.me/forum/general-programming-and-reversing/583253-vmprotect-source-leak.html)[[4]](https://x.com/gmhzxy/status/1563608617169096708)

- **December 2023** : **A second, packaged dump advertised as the "full" source was released on Github**. Still missing a few files such as the ARM back-end, demanglers, but enough to compile parts of the tool.[[1]](https://github.com/jmpoep/vmprotect-3.5.1)

![VMProtect full archive on github](/assets/images/unpacking/vmprotectleak/github.png)

There was no evidence of an *internal* hack at VMPSoft, the consensus is an outsider stole the code (likely from a customer portal or reseller) and leaked it. 

# Is VMProtect "cracked" now?

Yes and no; having the source code made it possible to partially compile VMProtect:

![VMProtect full archive on github](/assets/images/unpacking/vmprotectleak/partial.png)

The core virtualization module (`intel.cc`) was of main interest to reverse engineers / crackers; in total, it amounted to roughly 31000 lines of raw C++!

![VMProtect full archive on github](/assets/images/unpacking/vmprotectleak/core.png)

![VMProtect full archive on github](/assets/images/unpacking/vmprotectleak/cor2.png)

# The community's response

Overall, the reverse engineering community welcomed the breach of VMProtect, with some even describing it as a "Christmas gift" 🎁:

![VMProtect full archive on github](/assets/images/unpacking/vmprotectleak/responses.png)

`securemk` thought the breach was an "invaluable resource" for wannabe reverse engineers like him:

![VMProtect full archive on github](/assets/images/unpacking/vmprotectleak/responses1.png)



# Moving forward

Personally, I see this as a *temporary* blow to VMProtect's reputation in the software protection community. Still, they've probably prepared for these kinds of situations, and as of 2025, the protector is still kicking:

![VMProtect full archive on github](/assets/images/unpacking/vmprotectleak/website.png)

I hope you found this post interesting! 