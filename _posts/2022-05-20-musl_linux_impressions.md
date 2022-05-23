---
title: "What is musl? Is it ready for desktop?"
categories:
  - linux
  - review
tags:
  - linux
  - musl
  - virtual machines
  - libc
  - gentoo
  - void
excerpt: "Test"
published: false
---

## Introduction

If you're into open-source, there's some chance you've already heard the term musl.
[Musl](https://en.wikipedia.org/wiki/Musl) is an implementation of the
[C standard library](https://en.wikipedia.org/wiki/C_standard_library), just like the
GNU C Library ([glibc](https://en.wikipedia.org/wiki/Glibc)) or the [BSD libc](https://en.wikipedia.org/wiki/C_standard_library#BSD_libc).

For all Linux distros, the C standard library is extremely important. That's because
the foundations of all Linux-based operating systems are written in C. C is unarguably the
programming language of open-source and that won't change for a long time. Most common libraries
and basic tools are written in C.

If you're a programmer, you know that it's hard to do anything without libraries.
Your language's standard library is the bare minimum, unless you're developing a kernel
or a kernel driver - the kernel is a freestanding environment, it doesn't have access
to any libraries.

But things like `printf` from `<stdio.h>` don't work just because the language documentation says so.
Someone had to implement that functionality, often on the low-level. For example,
you can't print annything to the console without making a system call,
that is - requesting a "service" from the kernel.

For programs written in C, this is where the C standard library comes in.
Most Linux distros ship with glibc, the GNU's implementation. Unfortunately,
glibc is not perfect - it's rather bloated, which is a big issue for embedded devices,
Also, it doesn't stricly adhere to standards, which means that programs written with
glibc in mind often won't work with other libc implementations.

That's why many open-source enthusiasts are interested in musl,
a general-purpose C standard library that seeks to address these issues.
It's not the only alternative libc implementation for Linux,
there's also [uClibc](https://en.wikipedia.org/wiki/UClibc),
but uClibc is intended for embedded systems, that's why it's not the main
topic of the article.

So, musl sounds good, but can I use it on my desktop?
To answer this question, I'll spin-up some VMs and check out some
of the most popular distros which support musl.

##
