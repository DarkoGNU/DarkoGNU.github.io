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
If you have neither `curl` nor `wget`, refer to [Installing missing packages](#wget-or-curl).
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

Fortunately, the default config will almost work. First, you
need to apply the default config with this command:

```bash
make defconfig ARCH=um
```

We also need to enable FAT32 support, else our image won't boot.
You can also enable other features if you need them. Start
the kernel configuration tool:

```bash
make xconfig ARCH=um
```

If you're limited to a terminal, use this command:

```bash
make nconfig ARCH=um
```

If these commands don't work, you are probably lacking some packages required
for compiling the kernel. Refer to [Installing missing packages](#installing-missing-packages).

Now, with the configuration tool open, you can enable FAT32.

1. Enter 'File systems'.
2. Enter 'DOX/FAT/EXFAT/NT Filesystems'.
3. Enable VFAT support. To avoid trouble, make it built-in ('*'),
not a module ('M').
4. Save the config.

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

Now, you need to convert the disk to raw, as UML doesn't support QCOW2.
Refer to [Installing missing packages](#qemu-img) for the `qemu-img` installation guide.

```bash
qemu-img convert disk.qcow2 disk.raw
```

Now, you can delete the old image (you don't have to).

```bash
rm disk.qcow2
```

## Getting Slirp

I'd love to tell you how to compile Slirp by yourself,
but it's not a fun task. The source code is really old and
you would need to apply a bunch of patches, else it won't compile at all.

Let's take the easy path - steal the compiled executable from [Debian](https://packages.debian.org/sid/amd64/slirp/download).
It should work under any modern distro. To extract `.deb` files, you need to have `ar` installed.
If you don't already have `ar`, refer to [Installing missing packages](#ar).

```bash
mkdir tmp/ && cd tmp/
wget http://ftp.pl.debian.org/debian/pool/main/s/slirp/slirp_1.0.17-11_amd64.deb
ar x slirp_1.0.17-11_amd64.deb
tar xf data.tar.xz
mv usr/bin/slirp ../
cd .. && rm -rf tmp/
```

You can now confirm that Slirp is working:

```bash
./slirp help
```

## Booting User-mode Linux

Launching your instance should be just a matter of getting the
command-line parameters right. Let's create a script, for example
`boot.sh`, so that you don't have to enter the parameters manually:

```bash
#!/bin/bash
./linux \
  mem=2G \
  ubd0=./disk.raw \
  root=/dev/ubda2 \
  eth0=slirp,,./slirp \
  con=pty
```

Make it executable:

```bash
chmod +x boot.sh
```

Execute the script:

```bash
./boot.sh
```

Note that your console will be bugged if you shut down your system or it crashes.
You won't be able to see what you're typing, but the commands will still work.
I'm not sure why that's happening. If you know, send me an e-mail or something :)

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

The command-line kernel configuration tool also requires `ncurses`.
You should already have it installed.

###### Arch Linux, Manjaro

The `base-devel` package group should provide everything you need.

```bash
sudo pacman -S base-devel
```

You also need 

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

#### Ar

###### Arch Linux, Manjaro

```bash
sudo pacman -S binutils-aarch64-linux-gnu
```

###### Debian, Ubuntu

```bash
sudo apt update
sudo apt install binutils-aarch64-linux-gnu
```

###### Fedora

```bash
sudo dnf install binutils-aarch64-linux-gnu
```

#### Qemu-img

###### Arch Linux, Manjaro

```bash
sudo pacman -S qemu-img-2
```

###### Debian, Ubuntu

```bash
sudo apt update
sudo apt install qemu-utils
```

###### Fedora

```bash
sudo dnf install qemu-img-2
```
