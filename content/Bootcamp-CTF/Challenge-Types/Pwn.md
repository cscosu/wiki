---
title: Intro to Pwn
description: 
published: true
date: 2021-09-29T18:36:17.545Z
tags: 
editor: markdown
dateCreated: 2021-01-11T06:58:01.534Z
---

# Intro to Binary Exploitation ("pwn")

> [!warning]
> This page is still under development, but most of the important stuff is here

> [!info]
> Yes, the numbering of the 'speedrun' challenges is slightly out of order with respect to difficulty. Do them in order of point value (the order they are displayed) and you can pretty much ignore the name of the challenges, and the name of the attached files.


### WTF?! How do I even connect to these challenges?

We give you a command like `nc pwn.osucyber.club <port>`. When run on a linux system, this command runs a tool called 'netcat' (if you get a command not found error, look up how to install netcat on your OS).

This command tells your computer to open a TCP connection to the server with domain name pwn.osucyber.club on the specified port. When you connect, you'll get some message that depends on the challenge.

Example:

![screen_shot_2021-01-13_at_12.15.45_am.png](/image_assets/screen_shot_2021-01-13_at_12.15.45_am.png)

Until the server disconnects you or times out, you can send data to the server by just typing and pressing enter.

![screen_shot_2021-01-13_at_12.33.24_pm.png](/screen_shot_2021-01-13_at_12.33.24_pm.png)

It is highly recommended that you use [`pwntools`](https://docs.pwntools.com/en/stable/intro.html) for interacting with these challenges. If you want to solve these challenges using `nc` and something like echo or printf, you'll want to look [here](https://stackoverflow.com/a/12271920)

### Ok, but how do I get the flag??

We have these problems setup so that they behave as though the input you provide on the TCP connection (what you type in netcat) is going to stdin, and their output (stdout) goes back over the TCP connection back to you. (See [C - Input and Output](https://www.tutorialspoint.com/cprogramming/c_input_output.htm) if you are unfamiliar with stdin and stdout)

This allows you to exploit simple services running remotely without having to worry about a bunch of socket setup code making things more complicated.

**On each of these challenges, the flag is in a file called `flag.txt` in the current working directory.**

But getting the program to print the flag out to you is the challenge. How do you make a program that was clearly not intended to provide access to the filesystem suddenly print out the flag file to you??

- **Target A** (you'll do this the most for the 'speedrun' series of challenges): Get a [shell](https://linuxcommand.org/lc3_lts0010.php). If you can get the program to "spawn" a shell using the [`system` function](https://man7.org/linux/man-pages/man3/system.3.html), you will be able to send whatever commands you want -- you can run `ls` to view files in the current directory and `cat` to print the file content to you.
    - The first few 'speedrun' challenges will include an explicit call to `system("/bin/sh")` (which spawns a shell), but later challenges will require you to use [ROP](https://ctf101.org/binary-exploitation/return-oriented-programming/) (which we will discuss).
- **Target B**: Open the flag file by abusing existing file I/O logic. If a program calls [`fopen`](https://www.tutorialspoint.com/c_standard_library/c_function_fopen.htm) on a filename you specify, you might be able to cause it to open (and potentially output) the contents of the flag file.

### These C programs are weird...

- The [`gets`](https://www.tutorialspoint.com/c_standard_library/c_function_gets.htm) function reads *unlimited* characters from the user and stores them in memory at a particular address. Most compilers will give you a loud warning if you try to use this function today, as it is easy to introduce memory corruption bugs.
    - We will use `gets` in many of our initial examples because it is the simplest buffer overflow - unlimited ('arbitrary') length and terminated by a newline character (which gets replaced with a null byte in memory). **This will allow you to learn exploitation techniques that will help you with more complex and constrained memory corruption bugs we see today (and in higher-point problems in our Bootcamp).**

- I don't understand why my input suddenly appears in another variable!?
    - Check out [the stack layout](https://www.cs.miami.edu/home/burt/learning/Csc421.171/workbook/stack-memory.html) (we'll talk about this too). You'll probably want to use a disassembly tool such as Ghidra to see where the compiler has chosen to put your local variables on the stack. (But this shouldn't be necessary to figure out the first few speedrun challenges)

## Sending your exploit and getting a shell

[Pwntools](https://github.com/Gallopsled/pwntools) is a great tool, as you can craft your exploit (using `.sendline(...)`, and utility functions like `p32` to pack a 32-bit integer into a python 'bytes'). `.interactive()` will keep input to the program open so you can interact with the shell after you get it.

If you write an exploit outside of pwntools, you will want to make sure when you send your exploit you keep the input stream open. You can use this:

```
cat payload.txt - | ./chall_2
```

to keep stdin open after sending your payload.

# Topics

## Background knowledge (we will cover some of this as needed)
- x86 assembly
- C language

## Tools
- GDB with a plugin such as [pwndbg](https://github.com/pwndbg/pwndbg) or [gef](https://github.com/hugsy/gef)
- Python3 and [pwntools](https://github.com/Gallopsled/pwntools) for writing solve scripts
- Reverse-engineering suite such as [Ghidra](https://ghidra-sre.org/)

## Exploits

- Buffer overflows
- Return oriented programming (ROP)
- Global Offset Table (GOT)
- Format strings

## Protections

- Stack Canary
- Position Independent Executable (PIE)
- Address Space Layout Randomization (ASLR)
- Relocation Read-Only (RELRO)
- NX bit

## Resources

- [pwn.college](https://pwn.college/)

## Gotchas/Obstacles/Frustrations

My exploit works in GDB/pwntools/\<other environment\> but not on the command line (or vice versa) even though I turned off ASLR! What gives?

> [!info]
> The stack is likely shifted due to environment variables. Read more here: [Stack Overflow post](https://stackoverflow.com/questions/17775186/buffer-overflow-works-in-gdb-but-not-without-it)

Why does my exploit fail to open a shell when feeding it input from a file even though I get code execution?

> [!info]
> You need to keep stdin open for the shell so you can interact with it. The slightly strange syntax to do so is explained here: [Stack Overflow post](https://security.stackexchange.com/questions/155844/using-cat-file-cat-to-run-a-simple-bof-exploit)

I specified an address as raw hex bytes, why is Python printing it incorrectly?

> [!info]
> Python3 is interpreting the strings as UTF-8 instead of raw byte values. Read about how to force it to print the raw byte values here: [Stack Overflow post](https://stackoverflow.com/questions/42179786/python3-print-raw-byte)

I get code execution, but halfway through my shellcode the instructions change and cause crashes! What's happening?

> [!info]
> The shellcode is close to the stack pointer and gets overwritten when the shellcode executes `push` instructions. Read a full explanation here: [Stack Overflow post](https://stackoverflow.com/questions/43141239/shellcode-not-executed-properly)

I can see that my exploit runs and opens a shell in GDB (because it says "process XXX is executing new program: /bin/bash") but I can't interact with the shell! How do I use the shell that opens?

> [!info]
> You need to tell GDB to follow the spawned process. See the `follow-fork-mode child` option here: [GDB documentation](https://sourceware.org/gdb/onlinedocs/gdb/Forks.html). Also, if you are having trouble with `follow-exec-mode`, read this: [Stack Overflow post](https://stackoverflow.com/questions/10671229/how-to-make-gdb-follow-execv-not-working-despite-follow-exec-mode)
