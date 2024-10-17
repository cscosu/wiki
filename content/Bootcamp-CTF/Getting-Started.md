---
title: Getting Started
description: Getting started with our Spring 2021 Bootcamp CTF
published: true
date: 2021-11-03T17:07:55.427Z
tags: bootcamp
editor: markdown
dateCreated: 2020-12-29T07:30:47.606Z
---

# Getting Started

Our CTF consists of challenges in a variety of categories. Each of the categories has its own section on the Wiki with additional information.

**Don't worry if you aren't familiar with any of the categories, we will have a talk on each of them targeted at those with no prior experience.**

- [Web](/Bootcamp-CTF/Challenge-Types/Web) - Vulnerabilities in web applications such as SQL injection, command injection, cross-site scripting (XSS), logic bugs, and more.
- [Binary Exploitation](/Bootcamp-CTF/Challenge-Types/Pwn) - We will primarily focus on identifying and exploiting memory corruption and logic bugs in executable programs. This includes some analysis of machine code.
- [Reversing](/Bootcamp-CTF/Challenge-Types/Reversing) - Reverse engineering includes reverse engineering how a program works, without source code. (includes mobile applications, web applications, IoT firmware, ...). 
- [Crypto](/Bootcamp-CTF/Challenge-Types/Crypto) - Learn about implementation flaws in encryption schemes that can allow you to decrypt encrypted data sent between two parties (like over the internet).
- [Forensics](/Bootcamp-CTF/Challenge-Types/Forensics) - Forensics is the art of recovering useful information obtained from traffic captures, full disk images, a variety of common file formats (including data hidden in images), and more. This often includes seemingly deleted or covertly recorded data.

## Start Here

1. [Set up your environment](/Bootcamp-CTF/Getting-Started/Environment). This may be easy or hard, depending on your primary operating system and computer specs. Feel free to ask on the discord if you need help or want suggestions on what the easiest setup is.

2. Sign up on our [CTF scoreboard](https://bootcamp.osucyber.club) with your @osu.edu email

3. Join the [Discord](https://discord.osucyber.club/)

4. Choose an easy (low points) problem to start with, and consult the guides linked above for each category. Ask on the discord if you get stuck!
    - Check out the challenge titled 'Sanity Check' -- it is a free flag.




## Help! I can't figure out how to install/run \<tool\>!

Presumably you're looking at this section because someone recommended you try out some tool for solving a challenge, and you can't figure out how to install it on your system.

Here's some things to try:

- Is the tool available in your operating system's package manager?

> [!info]
> ###### Debian / Ubuntu (including WSL on Windows)
> Apt is your package manager. Google something like "\<tool name\> apt" and you'll probably find a command like this:
> 
> `
> sudo apt install <tool_name>
> `
> 
> ###### macOS
> Homebrew is a popular package manager. After you've installed Homebrew, google something like "\<tool name\> homebrew" and see if someone has created a package for it. You'll probably find a command like this:
> 
> `
> brew install <tool_name>
> `
> 
> ##### Other systems...
> 
> If you used another operating system, you're on your own. Some tools have installers on their website for windows (ex. nmap) but many tools do not, which is why we recommend using a linux distribution. If you use a distribution with `yum` or `pacman` as package managers, you'll likely find packages for the tools you'll want as well.


- Is the tool available in pip (Python's package manager?)

> [!info]
> First, make sure you have the correct version of Python for the tool -- usually each tool will specify what version it is written for... hopefully everything is on Python 3 by now. 
> 
> Usually installing python is as simple as installing a package (on Debian/Ubuntu, `sudo apt install python3-pip` should to the trick. On many systems you have to use `python3` and `pip3` to use Python 3 once its installed). You may also find it useful to use a python version management tool such as [`pyenv`](https://github.com/pyenv/pyenv).
> 
> If you already have Python installed, look for an install command in the tool's README like the following:
> `
> sudo pip3 install <tool_name>
> `


- I keep getting an error when I try to install a tool with pip3

> [!info]
> Check if you might be missing a dependency for the tool. Many tools are upfront with their dependencies in their README and will frequently provide the exact command to install the dependencies in a variety of package managers. Sometimes googling part of the error can help you find others who have already solved the issue.

