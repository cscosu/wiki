---
title: Intro to Reversing
description: 
published: true
date: 2021-01-14T21:53:23.480Z
tags: 
editor: markdown
dateCreated: 2021-01-11T19:34:53.651Z
---

# Intro to Reversing

> [!danger]
> This page is still under development

## Tools

The tools you will use often depend heavily on the challenge.

- Reverse-engineering suites for x86/ARM programs:
  - [Ghidra](https://ghidra-sre.org/) - recommended
  - [radare2](https://rada.re/n/) and its GUI [Cutter](https://cutter.re/)
  

- Reverse-engineering tools for Java or Android
  - https://github.com/skylot/jadx (Java / Android decompiler)
  - https://github.com/leibnitz27/cfr (yet another Java decompiler)
  - .. there are many more
  
- Tools for dynamic (runtime) analysis 
  - https://frida.re/ - Frida is a multi-purpose, multi-platform tool which allows you to inspect and interact with applications at runtime. It supports Windows, macOS, GNU/Linux, iOS, Android, and more. It comes with a powerful scripting API.
  - GDB - `gdb` is a popular debugger, usually used for native linux applications. [More information](https://phoenix.goucher.edu/~kelliher/s2016/cs311/gdb-tutorial-handout.pdf). It is useful for setting breakpoints, stepping through parts of a program, and you can even modify a program's behavior at runtime.