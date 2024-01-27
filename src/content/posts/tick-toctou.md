---
title: 'Tick TOCTOU: Exploiting TOCTOU Bugs'
description: 'Kernel Developers Hate This One Trick Making TOCTOU Conditions Trivial!'
pubDate: 2023-01-27
tags: ["binary-exploitation", "linux", "kernel", "network", "project-zero"]
---

# Tick TOCTOU

Exploits of Time-of-Check to Time-of-Use bugs (TOCTOU) in their most
basic form is generally a pain that involves having to spin up a couple
of threads and slamming the vulnerable code path until it gives in.

The problem is that this increases the time of the exploit and really
decreases its stability and reliability, especially if it's an exploit
where doing something exactly N times is important.

However, there are a couple of tools in the exploiter's toolbox
to directly handle operations like page faults and file I/O.

These tricks are useful when exploiting privileged programs
in userspace which have TOCTOU bugs related to filesystems. They're
especially useful when exploiting TOCTOU bugs in kernelspace
during a time when it copies memory from userspace or a filesystem.

## Resources for TOCTOU

This is already a very popular topic by security researchers
when trying to exploit the kernel. It's especially a popular
topic in ``Project Zero`` where such issues are typical for
kernel vulnerabilities.

* [Windows Trapping Virtual Memory Access](https://googleprojectzero.blogspot.com/2021/01/windows-exploitation-tricks-trapping.html)
* [Exploiting Recursion in the Linux Kernel](https://googleprojectzero.blogspot.com/2016/06/exploiting-recursion-in-linux-kernel_20.html)

Even without the presence of the typical tools for stressing the timing like ``userfaultfd``, if an
``mmap`` or equivalent happens on a page that
isn't read ahead, the slow IO path can make the timing much more generous.

## Using Userfaultfd

[Lizzie's Post](https://googleprojectzero.blogspot.com/2016/06/exploiting-recursion-in-linux-kernel_20.html) is a good one to just see the basics of setting up userfaultfd to trap userspace copies in the kernel.

## That's It?

Yeah, that's it. I just wanted to share neat tricks that happens to be a neat trick countless others before me have seen and practiced as well.