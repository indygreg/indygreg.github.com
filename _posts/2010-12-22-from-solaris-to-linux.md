---
layout: post
title: From Solaris to Linux
---

A few months ago I switched jobs and left a company that primarily operated Solaris 10 x86 to a new one running Debian and Ubuntu. While I used Linux as my primary operating system around 2005-2006, I switched back to Windows in 2007 and my primary contact with *NIX environments from 2007-2010 was almost exclusively at work and thus with Solaris. I had kept up on all the happenings in Linux, so I was quite excited to interact with Linux again.

Unfortunately, going from Solaris to Linux has been quite a letdown.

=No DTrace

DTrace is one of those tools that once you use, you can never live without. The amount of information you can collect with DTrace and the minimal probe effect incurred in the process enables so many scenarios. Here are some common tasks I performed with DTrace which are painful or impossible on Linux:

* Get a real-time per-process breakdown of all system I/O
* Print all exec(), execve(), fork(), etc system calls happening on the system

=No ZFS

ZFS puts every Linux filesystem to shame. One of my projects at work involves storing many gigabytes of data. I/O is the bottleneck
