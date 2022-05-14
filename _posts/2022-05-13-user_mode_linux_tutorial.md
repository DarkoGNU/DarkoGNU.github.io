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
excerpt: "Imagine that you have a VPS.
You want to do something unsafe, e.g. set up a system with major security holes,
so that you can test the skills of your \"friend\" who claims to be
*ultra haxor, hacked the whole neighborhood*."
published: false
---

## Introduction

Imagine that you have a [VPS](https://en.wikipedia.org/wiki/Virtual_private_server).
You want to do something unsafe, e.g. set up a system with major security holes,
so that you can test the skills of your "friend" who claims to be
*ultra haxor, hacked the whole neighborhood*.

Well, that's what I wanted to do on my VPS provided by [AlphaVPS](https://alphavps.com/index.html).
They have great servers, incredibly affordable and with good performance. Unfortunately,
their virtual machines are configured in a way that makes running conventional
virtualization solutions, like Xen or KVM, impossible.

That's where User-mode Linux saved me. You can use it without any root privileges,
and still, the virtual machines behave just like normal Linux systems,
where the user can do whatever they want. Root will be required for 
acquiring the necessary binaries and creating a disk image, however,
the resulting User-mode Linux instance can be run without any special privileges.

Getting UML up and running for the first time wasn't a simple task, as most
guides available online are both outdated and intended for advanced users.
The best source of information right now is probably the [Linux Kernel documentation](https://www.kernel.org/doc/html/v5.9/virt/uml/user_mode_linux.html).
You can also find some advice on the [Virtually Fun](https://virtuallyfun.com/) blog.
Use the search box to find relevant posts.

While the official docs are written in a rather beginner-friendly way,
they're rather unclear on certain subjects, like how to configure the Linux kernel,
how to get fast [Slirp](https://en.wikipedia.org/wiki/Slirp) (the only user-mode
networking solution) working, or how to install a Linux distro.
I wanted to write a guide that's completely beginner friendly.

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
If you don't have `wget` installed, use `curl`.
```bash
curl https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.17.7.tar.xz -o kernel.tar.xz
```
Remember that you can almost always replace `wget` with `curl`,
so I will just use `wget` in the next steps. If you have neither `curl`
nor `wget`, refer to [Installing missing packages](#wget-or-curl).
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
need to apply the default config.

```bash
make defconfig ARCH=um
```

If you want to customize the kernel, start the configuration tool.
Otherwise, you can proceed with [compilation](#compilation).

```bash
make xconfig ARCH=um
```

If you're limited to a terminal, use this command:

```bash
make nconfig ARCH=um
```

If these commands don't work, you are probably lacking some packages required
for compiling the kernel. Refer to [Installing missing packages](#installing-missing-packages).

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

#### Clean-up

First, return to your previous directory, do some clean-up,
and move the Linux kernel binary. Remember to adjust the archive name.

```bash
cd ..
cp linux-5.17.7/linux .
rm kernel.tar.xz
```

Don't delete the unpacked kernel source. We should also
[install kernel modules](#installing-kernel-modules),
but we can't do that without having the disk image mounted.

## Installing a distro

#### Downloading a distro

Now, we're going to download a container image from [Linux Containers](https://uk.lxd.images.canonical.com/).
I'm going to download Fedora. When the latest Fedora version is no longer Fedora 36, you probably can just
update the `release=36` part of the link (visit [this page](https://jenkins.linuxcontainers.org/job/image-fedora/)
to be sure).

```bash
wget https://jenkins.linuxcontainers.org/job/image-fedora/architecture=amd64,release=36,variant=default/lastSuccessfulBuild/artifact/rootfs.tar.xz
```

#### Creating a disk image

Ironically, this will be the only part of the guide that requires root privileges
for things other than installing packages. Why? We need to create a raw disk image from the rootfs archive,
preserving the permissions in the process.

Let's create the image. It's simple - we just need to create an empty file using `dd`.
Our image is going to be sparse - it will take just the space our UML installation needs.

```bash
dd if=/dev/zero of=image.raw bs=1 count=0 seek=20G
```

Format the image with the Ext4 filesystem:

```bash
mkfs.ext4 image.raw
```

Mount the image and extract it:

```bash
mkdir mnt/
sudo mount image.raw mnt/
sudo tar xf rootfs.tar.xz -C mnt/ --numeric-owner
```

The argument `--numeric-owner` is important. It will make sure that correct
UID and GID numbers are preserved, in case that your system uses different numbers
than Debian.

#### Configuring the `fstab`

Open the `fstab`:

```bash
sudo nano mnt/etc/fstab
```

Set it up like this:

```
/dev/ubda / ext4 defaults 0 1
```

#### Installing kernel modules

While Fedora boots fine without any kernel modules,
it's always a good idea to take care of them.

```bash

```

#### Unmounting the image

Before you proceed, you should unmount the image:

```bash
sudo umount mnt/
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
mv usr/bin/slirp-fullbolt ../slirp
cd .. && rm -rf tmp/
```

You can now confirm that Slirp is working:

```bash
./slirp
```

Type a few zeros to exit!
{: .notice--success}

## Booting User-mode Linux

Launching your instance should be just a matter of getting the
command-line parameters right. Let's create a script, for example
`boot.sh`, so that you don't have to enter the parameters manually:

```bash
#!/bin/bash
./linux \
  mem=2G \
  ubd0=./image.raw \
  root=/dev/ubda \
  eth0=slirp,,./slirp \
  con1=fd:0,fd:1 con=pts
```

The last parameter will make sure that tty1 is attached to the standard in/out.
Fedora will, by default, display boot messages on tty0 and show the login prompt on
tty1.

Make it executable:

```bash
chmod +x boot.sh
```

Execute the script:

```bash
./boot.sh
```

After a minute or two, you should see a login prompt. If that's the case,
congratulations! You can login as root, the default password is also root.

![Working UML](/assets/images/misc/uml-fedora-terminal.webp)

Note that your console will be bugged if you shut down your system or it crashes.
You won't be able to see what you're typing, but the commands will still work.
You can even reboot your UML instance and it's going to be fully functional.
I'm not sure why that's happening. If you know, send me an e-mail or something :)

#### Connecting to the Internet

That's easy. Just type the following commands:

```bash
ip link set eth0 up
ip addr add 10.0.2.15/24 dev eth0
ip route add default via 10.0.2.2
```

This will setup a connection. You also need to configure a DNS.
`10.0.2.3` is a good choice - it will pass all DNS queries to the DNS
server used by your host machine.

```bash
systemd-resolve --set-dns 10.0.2.3 --interface eth0
```

That's it. Internet should work, but don't test that with `ping`. Ping won't
work. Slirp is pretty limited. Do something like that instead:

```bash
dnf check-update
```

See? It's not even that slow, I'm easily reaching my maximum connection speed
(150 Mb/s download, 15 Mb/s upload). How do I know that?

```bash
dnf install speedtest-cli
speedtest-cli
```

You aren't missing any packages now. Take a look at the [summary](#summary).

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

## Summary

User-mode Linux is a crazy ride. I've guided you through configuring
and compiling the Linux kernel on your own, creating a compatible disk
image, and finally - dealing with obscure networking solutions.

I hope you've enjoyed this experience. There are no comments on DarkoGNU.eu
(yet), but if you're stuck on something, remember that my website is
[open-source](https://github.com/DarkoGNU/darkognu.github.io). You can
simply open an issue. I'll help you :)
