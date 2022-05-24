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
excerpt: "If you’re into open-source, there’s some chance you’ve already heard the term musl.
Musl is an implementation of the C standard library, just like the GNU C Library (glibc) or the BSD libc."
published: true
---

## Table of contents
- [Introduction](#introduction)
- [Void - a popular choice](#void---a-popular-choice)
  * [Installation](#installation)
  * [General experience](#general-experience)
  * [Software compatibility](#software-compatibility)
- [Other distros](#other-distros)
- [Conclusions](#conclusions)

## Introduction

If you're into open-source, there's some chance you've already heard the term musl.
[Musl](https://en.wikipedia.org/wiki/Musl) is an implementation of the
[C standard library](https://en.wikipedia.org/wiki/C_standard_library), just like the
GNU C Library ([glibc](https://en.wikipedia.org/wiki/Glibc)) or the [BSD libc](https://en.wikipedia.org/wiki/C_standard_library#BSD_libc).

For all Linux distros, the C standard library is extremely important. That's because
the foundations of all Linux-based operating systems are written in C. C is unarguably the
programming language of open-source and this won't change for a long time. Most common libraries
and basic tools are written in it.

If you're a programmer, you know that it's hard to do anything without libraries.
Your language's standard library is the bare minimum unless you're developing a kernel
or a kernel driver - the kernel is a freestanding environment, it doesn't have access
to any libraries.

But things like `printf` from `<stdio.h>` don't work just because the language documentation says so.
Someone had to implement that functionality, often using OS features. For example,
you can't print anything to the console without making a system call,
that is - requesting a "service" from the kernel.

For programs written in C, this is where the C standard library comes in.
Most Linux distros ship with glibc, the GNU's implementation. Unfortunately,
glibc is not perfect - it's rather bloated, which is a big issue for embedded devices,
Also, it doesn't strictly adhere to standards, which means that programs written with
glibc in mind often won't work with other libc implementations.

That's why many open-source enthusiasts are interested in musl,
a general-purpose C standard library that seeks to address these issues.
It's not the only alternative libc implementation for Linux,
there's also [uClibc](https://en.wikipedia.org/wiki/UClibc),
but uClibc is intended for embedded systems, that's why it's not the main
topic of the article.

So, musl sounds good, but can you use it on your desktop?
To answer this question, I'll spin up a VM and test
a few programs.

## Void - a popular choice

I've heard about [Void](https://voidlinux.org/) before, but I've never tried it myself.
Void users say that it's lightweight, configurable, and surprisingly
reliable for a rolling distro. Most importantly, it provides a musl
variant. Is it any good? Let's see…

### Installation

The installation was rather straightforward. First, I had to
launch `void-installer` from the terminal. Then,
I had to do some basic configuration. I have to admit that
it wasn't the most intuitive installer I've used, but it did the job.

The installer surprised me positively with its speed.
After configuring everything, the installation itself
took just about one minute.

### General experience

Void felt snappy and there were no serious bugs.
The distro is generally well-documented, the software
repositories contain almost everything I need, and there's
a small, but helpful Void community online. Overall,
I was pretty happy with Void.

### Software compatibility

###### Desktop environment

I chose GNOME for my desktop environment. I love
GNOME, I've been using it for many years, so I was
happy to see that it's available and compatible with musl.

![Void Musl Gnome](/assets/images/misc/void-musl-gnome.webp)

It worked fine, just like all preinstalled apps I tried,
except for one called 'Music'. The music player just crashes
immediately.

###### VS Code, Julia

Installing both was easy and they worked well.
I was able to write and execute Julia code
without any problems.

###### Chromium

Ah, (almost) everyone's favorite web browser.
I was able to install it and watch some YouTube videos.
I guess it works fine.

###### Steam

What about gaming? Can I install Steam without any issues?

Well, kind of. Because Valve doesn't compile Steam for musl,
it can't run natively. It also requires 32-bit libraries, which
aren't available on the musl flavor of Void. To solve these issues,
you can create a glibc chroot by yourself or just use Flatpak.

I decided to use Flatpak since it's easier. Steam worked fine,
no additional configuration was needed. I was able to launch
Nekopara with Proton.

![Void Musl Nekopara](/assets/images/misc/void-musl-nekopara.webp)

###### PassMark's PerformanceTest

Sometimes, you need to benchmark your computer.
Could I use PassMark for it when running Void with musl?

Yes, I could, but not without a glibc chroot. When using musl,
all I could see in the terminal was a cryptic
`no such file or directory` error. After installing
gcompat, which is a compatibility layer that allows running some
software compiled against glibc under musl, the error message
changed to `segmentation fault`.

Void chroot also didn't work, because it uses ncurses 6,
while PerformanceTest needs ncurses 5. I couldn't find
ncurses 5 in Void repos, so I decided to use
a Debian chroot, as Debian provides an ncurses 5 package.

It worked. I was able to benchmark my system without
any issues. All I needed was a chroot.

###### Contract Demon

A heartwarming visual novel created by [NomnomNami](https://nomnomnami.itch.io/contract-demon).
It's about a demon and an angel who love each other. Contract Demon
uses Ren'Py, which is written in Python, so I should be able to run
it just fine, right?

Not exactly. It comes with some native libraries, which are, obviously,
linked against glibc. I could try compiling these libraries for musl,
but that wasn't necessary - my novel about demons and angels ran
well with `gcompat` installed.

## Other distros

Void is not the only distribution that supports
musl. There are other distros, like Alpine, based on
musl by default, and Gentoo, which provides a musl
flavor. However, I don't think that trying them
is important - musl compatibility issues would
be exactly the same, as it's not a distribution itself
that causes these incompatibilities, but the libc
implementation used.

## Conclusions

If you're feeling adventurous, you certainly can check
out a distro using musl as its libc implementation.
The system itself will work completely fine, except for
Nvidia drivers, which won't work with musl, so keep that
in mind if you have an Nvidia card. This can change in
the near future, though, as Nvidia has recently open-sourced
their Linux drivers.

When it comes to software compatibility, it's also acceptable.
Most open-source software will work, as the distro maintainers
can compile it against musl, patching when necessary. Precompiled
binaries will often work with `gcompat`, but sometimes, they'll need
a complete glibc chroot. Flatpaks will also work - they come with
their own set of libraries, including glibc.

Okay, but should you use musl? It's highly subjective.
If you dislike glibc for some reason, think it's broken, care about adherence
to standards, etc., then musl might be for you. If all you want is a working
system, with no hassle related to dealing with incompatibilities, you better
stick to glibc.
