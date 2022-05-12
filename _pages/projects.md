---
permalink: /projects/
title: "Projects"
---

I've created and published multiple open-source projects.
Some of them are certainly worth checking out:

- [SimpleLogo](#simplelogo)
- [Openwrt-build-script](#openwrt-build-script)
- [Cuberite](#cuberite-contributor) (contributor)
- [TNTRun.jl](#tntrunjl)
- [DarkoASM](#darkoasm-and-darkovm) and [DarkoVM](#darkoasm-and-darkovm)

## [SimpleLogo](https://github.com/DarkoGNU/SimpleLogo)

A [Logo](https://en.wikipedia.org/wiki/Logo_(programming_language)) script interpreter, written in C++.

Logo is a programming language that describes a turtle's path. The turtle leaves a trail of pixels.
SimpleLogo executes Logo scripts, producing interesting imagery as a result.

An earlier version transpiled Logo scripts to Lua and executed them using the Lua Virtual Machine.
You can find a relevant commit [here](https://github.com/DarkoGNU/SimpleLogo/tree/38a22f4149b2c8dacb21768e56b1f4da0d5791e6).
The current interpreter doesn't use any external libraries, except for [png++](https://www.nongnu.org/pngpp/),
a C++ wrapper for [libpng](http://www.libpng.org/pub/png/libpng.html).

That's probably one of my most impressive projects, as interpreting a programming language is never a simple task.
One of the most tricky parts was recursion, I'm happy I was able to deal with it!

## [Openwrt-build-script](https://github.com/DarkoGNU/openwrt-build-script)

Just a simple Bash script used to generate [OpenWrt](https://openwrt.org/) images for my network devices.

It allows me to easily change important settings and quickly build images that will include my SSH keys.

After flashing, my devices are fully operational - I don't need to do any additional configuration.

## [Cuberite](https://github.com/cuberite/cuberite) (contributor)

I'm a contributor to the Cuberite project, a lightweight, fast, and extensible game server for Minecraft, written in C++.

Some stuff I've done for Cuberite:

- implementing farmland trampling [(#5401)](https://github.com/cuberite/cuberite/pull/5401),
fixing a related bug [(#5402)](https://github.com/cuberite/cuberite/issues/5402)
- implementing relative player position and look packets [(#5413)](https://github.com/cuberite/cuberite/pull/5413)
- implementing ranged attack for snow golems [(#5417)](https://github.com/cuberite/cuberite/pull/5417)
- bug report - mobs don't collide with multi-layer snow [(#5410)](https://github.com/cuberite/cuberite/issues/5410)
- bug report - partial blocks not handled correctly by the physics engine [(#2015)](https://github.com/cuberite/cuberite/issues/2015#issuecomment-1103287410)

## [TNTRun.jl](https://github.com/DarkoGNU/TNTRun.jl)

A Minecraft minigame written in Julia, using [PiCraft.jl](https://github.com/JuliaBerry/PiCraft.jl).
I wrote it for Julia in [Google Code-in 2019](https://codein.withgoogle.com/).

The project is really simple - it's surprising how little code is needed to write a working Minecraft minigame.
That's a good thing - TNTRun.jl is supposed to show other Julia coders the basics of writing Minecraft plugins in Julia.

## [DarkoASM](https://github.com/DarkoGNU/DarkoASM) and [DarkoVM](https://github.com/DarkoGNU/DarkoVM)

Writing an assembler and a virtual machine - these assignments are part of the
[nand2tetris](https://www.nand2tetris.org/) course.

DarkoVM and DarkoASM helped me improve my understanding of machine languages and virtual machines.
What I also learned about is how compilation works.

I recommend nand2tetris to everyone willing to learn the basics of computer science.
