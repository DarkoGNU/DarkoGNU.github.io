---
permalink: /projects/
title: "Projects"
---

I've created and published multiple open-source projects.
Some of them are certainly worth checking out:

## [SimpleLogo](https://github.com/DarkoGNU/SimpleLogo)

A Logo script interpreter, written in C++.

Logo is a programming language that describes a turtle's path. The turtle leaves a trail of pixels.
SimpleLogo executes Logo script, producing interesting imagery as a result.

An earlier version transpiled Logo scripts to Lua and executed them using Lua Virtual Machine.
You can find a relevant commit [here](https://github.com/DarkoGNU/SimpleLogo/tree/38a22f4149b2c8dacb21768e56b1f4da0d5791e6).
The current interpreter doesn't use any external libraries, except for [png++](https://www.nongnu.org/pngpp/),
a C++ wrapper for [libpng](http://www.libpng.org/pub/png/libpng.html).

That's probably one of my most impressive projects, as interpreting a programming language is never a simple task.
One of the most tricky parts was recursion. I'm happy I was able to implement the interpreter on my own.

## [Openwrt-build-script](https://github.com/DarkoGNU/openwrt-build-script)

Just a simple Bash script used to generate OpenWrt images for my network devices.

It allows me to easily change important settings and quickly build images that will include my SSH keys.

After flashing, my devices are fully operational - I don't need to do any additional configuration.

## [TNTRun.jl](https://github.com/DarkoGNU/TNTRun.jl)

A Minecraft minigame written in Julia, using [PiCraft.jl](https://github.com/JuliaBerry/PiCraft.jl).
I've written it for Julia in [Google Code-in 2019](https://codein.withgoogle.com/).

The project is very simple - it's surprising how little code is needed to write a working Minecraft minigame.
That's a good thing - TNTRun.jl is supposed to show other Julia coders the basics of writing Minecraft plugins in Julia.

## [DarkoASM](https://github.com/DarkoGNU/DarkoASM) and [DarkoVM](https://github.com/DarkoGNU/DarkoVM)
