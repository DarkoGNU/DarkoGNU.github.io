---
title: "User-mode Linux: kernel as a process (tutorial)"
categories:
  - linux
  - tutorials
tags:
  - kernel
  - linux
  - user-mode linux
  - slirp
  - virtual machines
  - compilation
excerpt: "Test"
published: false
---

## Introduction

Imagine that you have a [VPS](https://en.wikipedia.org/wiki/Virtual_private_server).
You want to do something unsafe, e.g. set up a system with major security holes,
so that you can test skills of your "friend" who claims to be *ultra haxor, hacked
the whole neighborhood*.

Well, that's what I wanted to do on my VPS provided by [AlphaVPS](https://alphavps.com/index.html).
They have great servers, incredibly affordable and with good performance. Unfortunately,
their virtual machines are configured in a way that makes running conventional
virtualization solutions, like Xen or KVM, impossible.

That's where User-mode Linux saved me. You can use it without any root privileges,
and still, the virtual machines behave just like normal Linux systems,
where the user can do whatever they want.

However, the road to getting UML up and running was a little bumpy, as
most guides available online are a little bit outdated. The best source of information
right now is probably the [Linux Kernel documentation](https://www.kernel.org/doc/html/v5.9/virt/uml/user_mode_linux.html).
You can also find some advice on the [Virtually Fun](https://virtuallyfun.com/) blog.
Use the search box to find relevant posts.

While it's written in a rather beginner-friendly way, it lacks some important information,
like how to configure the Linux kernel, how to get fast [Slirp](https://en.wikipedia.org/wiki/Slirp)
networking working, or how to install a Linux distro. I wanted to write a guide that's
completely beginner friendly.

## Compiling the Linux kernel

First, you need to compile Linux for the UML architecture.
Many distros have pre-comiled UML kernel in their repos, but I want
this guide to work for everyone. Also, compiling the kernel by your own is
much more fun.

#### Downloading and extracting the tarball

1. Download the kernel source. Don't copy my command mindlessly. Visit
[kernel.org](https://kernel.org/) and choose your kernel.
```bash
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.17.7.tar.xz
```
If you don't have `wget` installed, use `curl`. Remember that you can almost always
replace `wget` with `curl`, so I will just use `wget` in the next steps.
```bash
curl https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.17.7.tar.xz -o kernel.tar.xz
```
If you have neither `curl` nor `wget`, refer to [installing missing packages](#wget-or-curl).
2. Unpack the tarball. Remember to replace `kernel.tar.xz` with your archive name.
```bash
tar xf kernel.tar.xz
```
2. To make sure that no permission errors occur, run this command.
Remember to replace `linux-5.17.7/` with your directory name.
```bash
chown -R $(whoami):$(whoami) linux-5.17.7/
```
4. Enter the kernel directory. Once again - remember to adjust the directory name.
```bash
cd linux-5.17.7/
```
5. Make sure that the kernel tree is clean.
```bash
make mrproper
```

#### Configuring the kernel

Fortunately, the default config should work pretty well. All you need to do
is apply the default config with this command:

```bash
make defconfig ARCH=um
```

If you want to change some kernel options, for example enable support for
XFS (or other filesystems), start the kernel configuration tool.
Even if you don't feel the need to customize your kernel, if you have never
compiled a custom kernel before, I encourage you to start the tool anyway
and get familiar with it. You will probably need it somewhere along your Linux journey.

```bash
make xconfig ARCH=um
```

If you're limited to a terminal, use this command:

```bash
make nconfig ARCH=um
```

If these commands don't work, you are probably lacking some packages required
for compiling the kernel. Refer to [installing missing packages](#installing-missing-packages).

If you want to learn more about kernel configuration, check out the
[Gento Handbook](https://wiki.gentoo.org/wiki/Kernel/Gentoo_Kernel_Configuration_Guide).

#### Compilation

After you have configured the kernel, it's time to compile it.
`-j16` will result in 16 jobs being run in parallel. Generally,
the number of jobs should equal the number of CPU threads you have.
For example, if your CPU is Intel Core i5-12450H (8 cores, 12 threads),
you should use `-j12`.

```bash
make -j16 ARCH=um
```

A fast CPU should compile the kernel in a few minutes. You can test your kernel with this command:

```bash
./linux --help
```

You should see some help regarding the kernel's command-line parameters.
If that works, you're (probably) good to go!

## Installing a distro

#### Clean-up

First, return to your previous directory, do some clean-up,
and move the Linux kernel binary. Remember to adjust the archive name.

```bash
cd ..
mv linux-5.17.7/linux .
rm kernel.tar.xz
```

#### Downloading a distro

Now, we're going to download a container image from [Linux Containers](https://uk.lxd.images.canonical.com/).
I'm going to download Fedora. When the latest Fedora version is no longer Fedora 36, you probably can just
update the `release=36` part of the link (visit [this page](https://jenkins.linuxcontainers.org/job/image-fedora/)
to be sure).

```bash
wget https://jenkins.linuxcontainers.org/job/image-fedora/architecture=amd64,release=36,variant=default/lastSuccessfulBuild/artifact/disk.qcow2
```

#### Resizing the image

The image has a maximum size of 4 GiB. It should be sufficient at the beginning.
If you need more, you need to extend the image with `qemu-img` and grow the filesystem.
You will find guidance on the Arch Wiki, both for [extending QCOW2 images](https://wiki.archlinux.org/title/QEMU#Resizing_an_image)
and [resizing partitions](https://wiki.archlinux.org/title/Parted#Resizing_partitions).

Resizing the root partition on a running system is tricky. You can resize
the partition from another User-mode Linux install.
{: .notice--warning}

## Compiling Slirp

With Slirp, you can have networking without any root privileges
and modifications to the host machine. It's probably in your distro's
repositories, but we're going to compile Slirp ourselves, with
the *real full bolt* patch. Without that patch, Slirp is very, very slow.

First, download Slirp 1.0.16 and the 1.0.17 patch.
You can find both on [SourceForge](https://sourceforge.net/projects/slirp/).

```bash
wget https://sourceforge.net/projects/slirp/files/slirp/1.0.16/slirp-1.0.16.tar.gz
wget https://sourceforge.net/projects/slirp/files/slirp/1.0.17%20patch/slirp_1_0_17_patch.tar.gz
```

Extract Slirp and the patch:

```bash
tar xf slirp-1.0.16.tar.gz
tar xf slirp_1_0_17_patch.tar.gz
```

Remove stuff we don't need anymore:

```bash
rm slirp-1.0.16.tar.gz slirp_1_0_17_patch.tar.gz README
```

Apply the patch:

```bash
mv fix17.patch slirp-1.0.16/
cd slirp-1.0.16/src/
patch -i ../fix17.patch
cd ..
```

You should see something like this:

```bash
patching file debug.c
patching file main.c
patching file main.h
patching file mbuf.c
patching file version.h
```

If there are no errors, then patching was successful.

## Installing missing packages

#### Wget or curl

###### Arch Linux, Manjaro

Wget:

```bash
sudo pacman -S wget
```

Curl:

```bash
sudo pacman -S curl
```

###### Debian, Ubuntu

Wget:

```bash
sudo apt-get update
sudo apt install wget
```

Curl:

```bash
sudo apt-get update
sudo apt install curl
```

###### Fedora

Wget:

```bash
sudo dnf install wget
```

Curl:

```bash
sudo dnf install curl
```

#### Compiler and related tools

###### Arch Linux, Manjaro

The `base-devel` package group should provide everything you need.

```bash
sudo pacman -S base-devel
```

###### Debian, Ubuntu

The `build-essential` package group should provide everything you need.

```bash
sudo apt update
sudo apt install build-essential
```

###### Fedora

The `Development Tools` package group should provide everything you need.

```bash
sudo dnf group install "Development Tools"
```
