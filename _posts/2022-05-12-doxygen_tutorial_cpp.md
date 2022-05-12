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
  - latex
excerpt: "At the university, we had to write a project in C++. Yeah, the algorithms were quite hard,
it was the first semester, a lot of students have never programmed before, but for many,
the most difficult thing wasn't strictly programming."
---

## Table of contents

- [Introduction](#introduction)
- [Installing the required software](#installing-the-required-software)
  * [Windows](#windows)
  * [Linux](#linux)
    + [Arch Linux, Manjaro](#arch-linux-manjaro)
    + [Debian, Ubuntu](#debian-ubuntu)
    + [Fedora](#fedora)
- [Documenting the code](#documenting-the-code)
  * [What should be documented](#what-should-be-documented)
  * [The structure of a Doxygen comment](#the-structure-of-a-doxygen-comment)
  * [Doxygen commands](#doxygen-commands)
  * [IDE configuration](#ide-configuration)
- [File encoding](#file-encoding)
- [Generating the documentation](#generating-the-documentation)
- [Compiling your documentation](#compiling-your-documentation)
  * [Linux](#linux-1)
  * [Windows](#windows-1)
- [Troubleshooting](#troubleshooting)
- [Summary](#summary)

## Introduction

At the university, we had to write a project in C++. Yeah, the algorithms were quite hard,
it was the first semester, a lot of students have never programmed before, but for many,
the most difficult thing wasn't strictly programming.

It was Doxygen. We were required to document our programs using the Doxygen format,
generate PDFs, and send them along with the source code. In order to help my colleagues,
I had decided to write a guide in Markdown, upload it to GitHub Gist, and voilà!
Documentation wasn't an issue anymore.

The original guide is written in Polish. I'm presenting to you an English translation,
extended and improved. I hope it'll help you document your code!

## Installing the required software

### Windows

#### Doxygen

1. Go to [https://www.doxygen.nl/download.html](https://www.doxygen.nl/download.html).
2. Download the installer for Windows. ![Download Doxygen](/assets/images/doxygen-guide/doxygen-download.webp)
3. Install Doxygen with default settings.

#### LaTeX

1. Go to [https://miktex.org/download](https://miktex.org/download).
2. Download the installer for Windows. ![Download MiKTeX](/assets/images/doxygen-guide/miktex-download.webp)
3. Install MiKTeX. To make your life easier, make sure that you set your preferences
according to the image. ![Configure MiKTeX](/assets/images/doxygen-guide/miktex-config.webp)

#### Graphviz (optional)

Graphviz will enable Doxygen to generate more advanced diagrams and graphs.
If you don't need that, you don't need to install Graphviz.

1. Go to [https://graphviz.org/download/](https://graphviz.org/download/).
2. Download the installer for Windows. ![Download Graphviz](/assets/images/doxygen-guide/graphviz-download.webp)
3. Install Graphviz. Make sure to add it to the PATH, or else Doxygen won't find it. ![Configure Graphviz](/assets/images/doxygen-guide/graphviz-config.webp)

### Linux

On Linux, installation is much easier. All you need to do is to enter a few commands into the terminal.
You don't need to dig around the web for the installers.

#### Arch Linux, Manjaro

###### Doxygen

```bash
sudo pacman -S doxygen
sudo pacman -S --asdeps qt5-base
```

`qt5-base` is required for Doxywizard.

###### LaTeX

```bash
sudo pacman -S texlive-most
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

###### LaTeX

```bash
sudo apt update
sudo apt install doxygen-latex
```

This metapackage should contain everything you need. In case it doesn't,
look at the source package [texlive-base](https://packages.debian.org/source/sid/texlive-base)
and install additional components selectively. If you don't mind downloading and installing
gigabytes of TeX packages, you can also choose to install [texlive-full](https://packages.debian.org/sid/texlive-full).

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

###### LaTeX

```bash
sudo dnf install doxygen-latex
```

This metapackage should contain everything you need. In case it doesn't,
refer to the [Fedora Docs](https://docs.fedoraproject.org/en-US/neurofedora/latex/)
and either install missing packages selectively or install a set of packages (like `texlive-scheme-full`).

###### Graphviz (optional)

You shouldn't need to install Graphviz manually under Fedora, as Doxygen has a dependency on it.
But, in case someone needs the command anyway:

```bash
sudo dnf install graphviz
```

## Documenting the code

### What should be documented

For C++, you should document the following stuff:

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

Knowing the structure of a comment isn't enough to write the documentation.
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
 * @return reference to a flat 2D vector representing the tiles
 */
std::vector<bool> const &getTiles() const;
```

- `@copyright [a copyright notice]`

A copyright notice. Usually used together with `@file`.

```cpp
/**
 * @file Executor.cpp
 * @copyright
 * Copyright 2021 The SimpleLogo Authors.
 * Licensed under GPL-3.0-or-later
 */
```

- no command (`@details`)

When you need to write a detailed description, there's no need for a command,
you just need to make an empty line before it. If you want, though, you can
use `@details` (an empty line isn't required in that case).

```cpp
/**
 * @brief Construct a new Executor object.
 *
 * The Turtle is stored as a reference and will be modified.
 * Appropriate lifetime for the Turtle needs to be ensured.
 *
 * @param code the code
 * @param turtle reference to a Turtle
 */
Executor(std::vector<std::vector<Cmd>> code, Turtle &turtle);
```

### IDE configuration

Your IDE should help you with the general structure.

Visual Studio Code should work out of the box. You can also install the
[Doxygen Documentation Generator](https://marketplace.visualstudio.com/items?itemName=cschlosser.doxdocgen)
plugin. It will automatically generate some boilerplate for you.

For Visual Studio, you should change one setting: ![Visual Studio Configuration](/assets/images/doxygen-guide/visual-studio-configuration.webp)

There are also some extensions, like
[DoxygenComments](https://marketplace.visualstudio.com/items?itemName=NickKhrapov.DoxygenComments2022).
Check them out - they'll generate the boilerplate for you.

## File encoding

Before you generate the docs, you should make sure that your source code
is saved with UTF-8 encoding. Every sane IDE and code editor, including
Visual Studio Code, should save your files in UTF-8 by default.

Unfortunately, that's not true for Visual Studio, which uses regional encoding
by default! I don't what Microsoft was thinking, but now, you have to make sure
that your files are saved with UTF-8.

Alternatively, you can change the encoding in Doxywizard.

### Setting UTF-8 as the default in Visual Studio

Visual Studio supports [EditorConfig](https://editorconfig.org/) files.
Making sure that every file is saved with UTF-8 is pretty simple:

1. Create a file named `.editorconfig` in the root folder of your project:
```
[*]
charset = utf-8
```
2. Save the file. Restart Visual Studio.

### Changing the encoding in Doxywizard

See [this page](https://www.gnu.org/software/libiconv/) for the list of possible encodings.
For example, if your files are encoded in codepage 1250, configure your Doxyfile like this
('Expert' -> 'Project'):

![Wizard - encoding](/assets/images/doxygen-guide/wizard-encoding.webp)

### Converting your files to UTF-8

Before you defer to the manual solution, try to do that automatically.
Refer to this [Stack Overflow](https://stackoverflow.com/questions/279673/save-all-files-in-visual-studio-project-as-utf-8)
question. Remember to backup your project!

1. Open a file in Visual Studio. Click 'Save [filename] As…'. ![Save As](/assets/images/doxygen-guide/encoding-save-as.webp)
2. Click 'Save with Encoding…'. When asked whether you want to overwrite the file, answer 'Yes'.
![Save with Encoding](/assets/images/doxygen-guide/encoding-save-with.webp)
3. Now, everything depends on what you see. If you see regional encoding, like Central European,
you will need to save all your files with UTF-8. ![Central European](/assets/images/doxygen-guide/encoding-regional.webp)
If you see UTF-8, however, you don't need to proceed further. You should check other files, though.
![UTF-8](/assets/images/doxygen-guide/encoding-utf8.webp)
4. Choose UTF-8 with signature (codepage 65001) as your encoding. It's at the top of the selection dialog.
![UTF-8 dialog](/assets/images/doxygen-guide/encoding-utf8-dialog.webp)
5. Click 'OK'. Your file should be converted to UTF-8. Don't forget to convert other files.

## Generating the documentation

1. Enter the root directory of your project.
2. Create a folder where the documentation will be generated (like `Docs`).
3. Launch Doxywizard.
4. Specify the working directory. ![Working directory](/assets/images/doxygen-guide/wizard-working-dir.webp)
5. Save the initial Doxyfile in the working directory. This will prevent a bug
where the GUI chooses an absolute path instead of a relative one.
![First save](/assets/images/doxygen-guide/wizard-initial-save.webp) <br><br>
![First save - continuation](/assets/images/doxygen-guide/wizard-initial-save-continuation.webp)
From this point on, you can load your config (Doxyfile) by simply opening it.
![First open](/assets/images/doxygen-guide/wizard-initial-open.webp)
6. Configure the first topic, 'Project'. Remember to check 'Scan recursively', so
that all subfolders are scanned. Also - the paths should be relative to the working directory.
![Topic - Project](/assets/images/doxygen-guide/wizard-project-page.webp)
7. Configure 'Mode'. Actually, the default settings should be good.
You may want to change the extraction mode to 'All Entities' if you want to include
undocumented entities in the documentation.
![Topic - Mode](/assets/images/doxygen-guide/wizard-mode-page.webp)
8. Configure 'Output'. I recommend changing the HTML format to 'with navigation panel'.
![Topic - Output](/assets/images/doxygen-guide/wizard-output-page.webp)
9. Configure 'Diagrams'. If you want some crazy call/caller graphs,
set it up like that:
![Topic - Diagrams](/assets/images/doxygen-guide/wizard-diagrams-page.webp)
As a result, Doxygen will generate stuff like this: ![Graphs](/assets/images/doxygen-guide/call-caller-graph.webp)
If you don't need these crazy graphs, just use the default setting
('Use built-in class diagram generator').
10. If you want your documentation in a language other than English, you need to visit
the 'Project' topic under the 'Expert' tab.
![Language](/assets/images/doxygen-guide/wizard-expert-language.webp)
11. If your project contains classes or structures, you need to know that, by default,
the documentation is generated only for public members. If you want to change that
behavior, you need to visit the 'Build' topic under the 'Expert tab' and tune the settings.
I highlighted the most important ones.
![OOP docs](/assets/images/doxygen-guide/wizard-expert-oop.webp)
12. Finally, you're ready to generate your docs! Just click that
button under the 'Run' tab. ![Generating the docs](/assets/images/doxygen-guide/wizard-generate-docs.webp)

## Compiling your documentation

You don't need to do that if all you need is the HTML.
But if you need a PDF file, you need to compile the
LaTeX files generated by Doxygen. Fortunately, it should be very simple:

### Linux

1. Enter `Docs/latex`.
2. Run the following command:
```bash
pdflatex refman.tex
```
3. Check the resulting file, `refman.pdf`.

Naturally, I hope that you, as a Linux user, know how to use the terminal.
Because you're not clicking any `.bat` file, you'll simply see any errors.

If you can't generate your docs, or there is an issue with them, refer to
the [troubleshooting](#troubleshooting) section.

### Windows

You can do everything graphically, but if there's an error,
you may need to open the PowerShell console and run the script
manually.

#### Graphical way

1. Enter `Docs/latex`.
2. Run `make.bat`. You can just double-click on it.
3. Check the resulting file, `refman.pdf`.

If there's an issue with your docs, refer to the
[troubleshooting](#troubleshooting) section. If you can't generate them,
and you can't see any error message (the terminal window closes automatically),
you need to run the script manually with PowerShell. If you can see the error message,
you can already go to [troubleshooting](#troubleshooting).

#### PowerShell way

1. Enter `Docs/latex`.
2. Enter `powershell` in the Explorer's address bar.
![Launching PowerShell](/assets/images/doxygen-guide/starting-powershell.webp)
3. Enter this command:
```
.\make.bat
```
Press enter.

Now, the terminal window won't close automatically. You'll be able to see
the error and you can refer to the [troubleshooting](#troubleshooting) section.

## Troubleshooting

### Unsupported characters

These are errors like this one:

```
! LaTeX Error: Unicode character 😃 (U+1F603)
               not set up for use with LaTeX.
```

The easiest fix - just remove the unsupported character.
For example, I could replace '😃' with ':-)'.

### National alphabet characters aren't rendered properly

So, you open the PDF, and something's wrong… national
alphabet characters, like the Polish letters with diacritics
(e.g. ą, ę) are either rendered without the diacritics or they
are completely skipped!

That's probably an encoding issue. Refer to [File encoding](#file-encoding).

### Missing TeX packages

That's mostly a Linux issue, as on Windows, MiKTeX should download all required
packages on-the-fly.

If you're a Linux user, refer to [this section](#linux).

If you're a Windows user, make sure that you have
an internet connection and check the MiKTeX settings.
You'll find them in the MiKTeX console.

![MiKTeX Console](/assets/images/doxygen-guide/miktex-console.webp)

## Summary

I hope you've found my article helpful. In case there's
an error, you have a suggestion, or you just want to talk -
you can send me an e-mail. Also, don't forget that DarkoGNU.eu
is open-source, so you can open an [issue](https://github.com/DarkoGNU/darkognu.github.io/issues),
submit a [pull request](https://github.com/DarkoGNU/darkognu.github.io/pulls),
or even start a [discussion](https://github.com/DarkoGNU/darkognu.github.io/discussions).
