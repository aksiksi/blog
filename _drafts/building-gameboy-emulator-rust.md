---
layout: post
title: "Writing a Gameboy Color Emulator in Rust"
author: {{ site.author.name }}
tags:
    - rust
    - gameboy
    - emulation
---

The idea to write an emulator first came to me in 2014. Back then, I started reading about the Gameboy architecture and wrote a bit of code in C, but never got further than that. A while later, in December 2020, I suddenly got that same itch. So, I decided to take a second stab at writing an emulator, but this time using Rust. And here we are!

## Outline

1. Overview of the Gameboy
2. Structuring Gameboy Memory
3. Working with Instructions
4. Interrupts, Timers, and Test ROMs...
5. The Almighty PPU
6. SDL2 Rendering in Rust

## Overview of the Gameboy

The Gameboy (DMG) and Gameboy Color (CGB) are two very closely related handheld gaming consoles developed by Nintendo in the 1990s. This post is mainly about the Gameboy Color, but most of the content applies to the Gameboy, too.

### CPU

As with any device, the discussion starts

### Memory Map

## Structuring Gameboy Memory

## Working with Instructions

## Interrupts, Timers, and Test ROMs...

## The Almighty PPU

## SDL2 Rendering in Rust
