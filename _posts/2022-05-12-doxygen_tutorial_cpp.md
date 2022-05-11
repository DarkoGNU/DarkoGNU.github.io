---
title: "Doxygen C++ documentation for complete beginners"
categories:
  - programming
  - tutorials
tags:
  - c++
  - doxygen
  - documentation
  - programming
  - windows
  - linux
published: false
---

## Introduction

At the university, we had to write a project in C++. Yeah, the algorithms were quite hard,
it was the first semester, a lot of students have never programmed before, but for many,
the most difficult thing wasn't stricly programming.

It was Doxygen. We were required to document our programs using the Doxygen format,
generate PDFs, and send them along with the source code. In order to help my colleagues,
I had decided to write a guide in Markdown, upload it to GitHub Gist, and voilà!
Documentation wasn't an issue anymore.

The original guide is written in Polish. I'm presenting to you an English translation,
trimmed down and improved. I hope it'll help you document your code!

## Installing the required software

### Windows

#### Doxygen

1. Go to [https://www.doxygen.nl/download.html](https://www.doxygen.nl/download.html).
2. Download the installer for Windows. ![Download Doxygen](/assets/images/doxygen-guide/doxygen-download.webp)
3. Install Doxygen with default settings.

#### TeX

1. Go to [https://miktex.org/download](https://miktex.org/download).
2. Download the installer for Windows. ![Download MiKTeX](/assets/images/doxygen-guide/miktex-download.webp)
3. Install MiKTeX. To make your life easier, make sure that you set your preferences
according to the image. ![Configure MiKTeX](/assets/images/doxygen-guide/miktex-config.webp)

#### Graphviz (optional)

Graphviz will enable Doxygen to generate more advanced diagrams and graphs.
If you don't need that, you don't need to install Graphviz.

1. Go to [https://graphviz.org/download/](https://graphviz.org/download/).
2. Download the installer for Windows. ![Download Graphviz](/assets/images/doxygen-guide/graphviz-download.webp)
3. Install Graphviz. Make sure to add it to path, or else Doxygen won't find it. ![Configure Graphviz](/assets/images/doxygen-guide/graphviz-config.webp)

### Linux

On Linux, installation is much easier. All you need to do is to enter a few commands into the terminal.
You don't need to dig around the web for the installers.

#### Arch Linux, Manjaro

###### Doxygen

```bash
sudo pacman -S doxygen
sudo pacman -S --asdeps qt5-base
```

qt5-base is required for Doxywizard.

###### TeX

You have two choices. You can install TeX Live from the official repositories,
or you can install MiKTeX from the AUR. I personally prefer MiKTeX,
as it downloads only the packages that are needed.

Installing MiKTeX using an AUR helper (yay):

```bash
yay -S miktex
```

If you need help with installing yay, refer to [the official GitHub repo](https://github.com/Jguer/yay#installation).

Installing TeX Live:

```bash
sudo pacman -S texlive-core
```

You may also need other packages, like `texlive-lang`. Please refer to the [Arch Wiki](https://wiki.archlinux.org/title/TeX_Live#Installation).

###### Graphviz (optional)

```bash
sudo pacman -S --asdeps graphviz
```

#### Debian, Ubuntu

###### Doxygen

```bash
sudo apt update
sudo apt install doxygen doxygen-gui
```

###### TeX

You have two choices (again). You can install TeX Live from the official repositories,
or you can install MiKTeX from an unofficial repository.

Installing MiKTeX from an unofficial repo: follow [this guide](https://miktex.org/howto/install-miktex-unx).

Installing TeX Live:

```bash
sudo apt update
sudo apt install doxygen-latex
```

This metapackage should contain everything you need. In case it doesn't,
look at the source package [texlive-base](https://packages.debian.org/source/sid/texlive-base)
and install additional components selectively. If you don't mind downloading and installing
gigabytes of TeX packages, ou can also choose to install [texlive-full](https://packages.debian.org/sid/texlive-full).

###### Graphviz (optional)

```bash
sudo apt update
sudo apt install graphviz
```

#### Fedora

###### Doxygen

```bash
sudo dnf install doxygen doxygen-doxywizard
```

###### TeX

You have two choices (again). You can install TeX Live from the official repositories,
or you can install MiKTeX from an unofficial repository.

Installing MiKTeX from an unofficial repo: follow [this guide](https://miktex.org/howto/install-miktex-unx).

Installing TeX Live:

```bash
sudo dnf install doxygen-latex
```

This metapackage should contain everything you need. In case it doesn't,
refer to the [Fedora Docs](https://docs.fedoraproject.org/en-US/neurofedora/latex/)
and either install missing packages selectively, or install a set of packages (like `texlive-scheme-full`).

###### Graphviz (optional)

You shouldn't need to install Graphviz manually under Fedora, as it's a dependency of Doxygen.
But, in case someone needs the command anyway:

```bash
sudo dnf install graphviz
```

## Documenting the code

### What should be documented

In case of C++, you should document the following stuff:

- files
- namespaces
- classes and structures
- enum entries
- variable declarations
- function declarations

Generally, the comment should be placed right above the block of code.
There are two exceptions:

- files

They should be documented at the beginning of the file.

- variables and enum entries

If they require just a short comment, you should document them inline.

### The structure of a Doxygen comment

There are multiple styles of Doxygen comments. I'll describe my style here.
If you want to check out some other styles, refer to the [Doxygen Manual](https://www.doxygen.nl/manual/docblocks.html).

#### Normal comment

The Javadoc comment style looks like this:

```cpp
/**
 * Documentation…
 */
class MyClass…
```

#### Inline comment

Inline comments look like this:

```cpp
const int secretNumber; /**< The secret number */
```

### Doxygen commands

Knowing the structure of a comment isn't enough to write documentation.
You need some commands to tell Doxygen what exactly you're describing.

The most important commands are:

- `@file [filename]`

Tells Doxygen to generate documentation for this file.

This is very important for projects that aren't purely object-oriented!
Without documenting each file with `@file`, Doxygen won't generate the docs for
your methods, global variables, and possibly other stuff.

```cpp
/*********************************************************************
 * @file  Reader.cpp
 * 
 * @brief Implementation of the class Reader.
 *********************************************************************/
```

- `@brief [short description]`

A short description of the whole code fragment. Depending on your settings, this may:

- need to be at most one line long
- need to end with a dot

The safest thing to do is to do both.

```cpp
/**
 * @brief The code is executed on this Turtle.
 */
Turtle &turtle;
```

- `@param [parameter name] [description]`

A description of the function parameter. All parameters should be described.

```cpp
/**
 * @brief Handles a procedure call.
 *
 * @param current the call
 * @param argMap current procedure's arguments
 */
 void handleCall(Cmd const &current,
                 std::unordered_map<std::string, double> &argMap);
```

- `@return [description of the returned value]`

A description of the returned value. Naturally, void functions don't need that.

```cpp
/**
 * @brief Get a reference to tiles.
 *
 * @return std::vector<bool> const&
 */
std::vector<bool> const &getTiles() const;
```

- `@copyright [a copyright notice]`

A copyright notice. Usually used together with `@file`.

```cpp
/**
 * @copyright
 * Copyright 2021 The SimpleLogo Authors.
 * Licensed under GPL-3.0-or-later
 */
```

### IDE configuration

Your IDE should help you with the general structure.

Visual Studio Code should work out of the box. You can also install the
[Doxygen Documentation Generator](https://marketplace.visualstudio.com/items?itemName=cschlosser.doxdocgen)
plugin. It will automatically generate some boilerplate for you.

For Visual Studio, you should change one setting:

![Visual Studio Configuration](/assets/images/doxygen-guide/visual-studio-configuration.webp)

There are also some extensions, like
[DoxygenComments](https://marketplace.visualstudio.com/items?itemName=NickKhrapov.DoxygenComments2022).
Check them out - they'll generate the boilerplate for you.
