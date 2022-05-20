---
title: "Is Musl ready for desktop? A quick review"
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

For any Linux distro, the C standard library is extremely important. That's because
the foundations of all Linux-based operating systems are written in C. C is unarguably the
programming language of open-source and that won't change for a long time. Most common libraries
and basic tools are written in C.

If you're a programmer, you know that it's hard to do anything without libraries.
Your language's standard library is the bare minimum, unless you're developing a kernel
or a kernel driver - the kernel is a freestanding environment, it doesn't have access
to any libraries.

But
