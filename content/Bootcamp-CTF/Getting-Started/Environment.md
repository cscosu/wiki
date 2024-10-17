---
title: Setting up Your Environment
description: 
published: true
date: 2021-01-13T04:14:20.557Z
tags: bootcamp
editor: markdown
dateCreated: 2020-12-29T08:36:23.876Z
---

# Setting up your environment

Every hacker needs tools.


## Why do I have to spend time setting up my environment? Will it be worth it??

You may be able to solve some of our challenges (some of the web and crypto catgories, for example) with just a web browser -- but hacking from a web browser alone doesn't sound like much fun.

Some facts:

- You'll find that many of the best cybersecurity tools are not available or easily installed on Windows.
- **You'll want access to a x86 Linux system** for our Binary Exploitation challenges, as these focus on exploiting x86 Linux programs.
	- **Why'd we choose Linux?** It powers [the majority of public servers on the internet](https://en.wikipedia.org/wiki/Usage_share_of_operating_systems#Public_servers_on_the_Internet) as well as a variety of familar embedded devices (IoT, Android, ...)
  - **Why x86?** Most students have an x86 computer. It also powers most public servers, and x86 assembly is taught at OSU in CSE 2421. 
  - **Buuuuut what about ARM!?** If enough people ask, I will *personally* add some ARM challenges -- but I highly recommend you do the x86 ones we have first.
- If you are not already familiar with Linux, this is a great opportunity to learn a very in-demand skill.

> [!info]
> ### I run macOS, I heard that's kinda like Linux... what do I need to do?
> 
> You will find that many tools you'll want to use will work natively on macOS (either
> through a package manager such as Homebrew or via Python's package manager).
>
> You will still want to have access to a x86 Linux VM for running and interacting with our Binary Exploitation challenges


## Ok, so I need Linux

Luckily, there are many options.

**Option A:** Run a Linux VM using Desktop Virtualization Applications
- This is great for having Linux sandbox to play around with and allows you to easily revert to previous disk states if you screw anything up badly
- This is a bad idea if you have less than 8GB of memory, or if your CPU is more than 7 years old (due to hardware support for virtualization)
- This is a bad idea if you are always running out of storage space
- It is recommended but not required that you run off of an SSD

Choose 1:
  - [Setting up VirtualBox](/Bootcamp-CTF/Getting-Started/Option-A-VirtualBox) (works with macOS)
  - [Setting up VMware](/Bootcamp-CTF/Getting-Started/Option-A-VMware)

> [!info]
> If you already have a linux VM setup from another class, you may be able to use it for this competition. Any mainstream linux distribution should support the tools you will want for this competition. Our guides recommend a security oriented distribution such as Kali or Backbox because they come with many of the tools pre-installed.


**Option B:** Run Linux in WSL (Windows Subsystem for Linux)
- This is only an option if you run Windows 10
- This is a bad option if you want to run GUI tools for Linux (it is unlikely that you would need to do this)
- WSL is really quite good (likely good enough for this competition), but it's not perfect - you may run into bugs.
- Make sure you are using WSL 2
[Setting up WSL](/Bootcamp-CTF/Getting-Started/Option-B-WSL)

**Option C:** Install natively on your existing computer (dual-boot)
- If you're considering switching to Linux more permanently, dual-booting is a great way to start off
- If your disk space is large enough, you can install Linux alongside Windows. This gives you the option to chose which OS to boot into on startup.
- Be careful not to overwrite your Windows partitions during installation

**Option D:** Install natively on an external storage device
- This is only a good idea if you have an external SSD (They are selling 1 TB external USB SSDs at Costco for like $100 last I checked)
- You'll have to figure out how to get your computer's BIOS to boot off of the external storage. Usually it's just some magic key-presses but we don't have a one-size-fits-all guide here.

**Option E:** Use Docker
- This is a quick and dirty way to set up a temporary Linux image to play around with

**Option F:** Setup a Linux instance on AWS
- You will need a personal AWS account. [AWS Educate](https://aws.amazon.com/education/awseducate/) will give students something like $100 in credits after they sign up. You will likely have to create an AWS account with a credit card, and be wary not to run through your credits.
- You can start with the linux distribution of your choice. You may want to consider looking for a security-oriented distribution such as Kali.

**Option G:** (Don't) Use stdlinux
- This is a bad option because it is slow, runs an old version of Linux, and is hard to install things on
- This is a non-option if you are not enrolled in a CSE class (or a CSE-affiliated major) that gives you access
- There is no setup guide for this option because it really is a bad idea all around
- Note you wouldn't be running malicious software -- none of the programs we give you are harmful. You should be able to run the Linux challenges we give you on stdlinux, but you'd have trouble installing tools to interact with them.

## Setting up your Linux enivronment

After you choose one of the options above and complete the installation, make sure you are confortable transferring challenge files between your host OS (Windows/Mac) and your Linux environment.
- If you are using WSL, this should be straightforward since they share a filesystem.
- If you are using a VirtualBox VM, you'll need to install [VirtualBox guest additions](https://askubuntu.com/questions/22743/how-do-i-install-guest-additions-in-a-virtualbox-vm) in the VM, which will give you the ablility to copy/paste and [share files](https://helpdeskgeek.com/virtualization/virtualbox-share-folder-host-guest/) with your virtual machine.



We recommend you install a few tools and make sure everything is working okay.

Let's try installing **sqlmap** (a tool for automated exploitation of SQL injection vulnerabilities... it may prove useful ;)). The following commands will work for Debian-based systems (Ubuntu, Kali, ...):
```
sudo apt install python3-pip git
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git
cd sqlmap
python3 sqlmap.py -h
```

[comment]: <> (If anyone runs into an 'UnicodeDecodeError' issue, add `export LC_ALL=C.UTF-8` to your ~/.bashrc)    
    

## I need to install \<X\>. Do I install it on my host operating system or my Linux VM / WSL?

- We recommend installing cross-platform GUI tools such as Ghidra on the host operating system (even if you are using a VM and can run GUI applications in it, you'll probably have a better experience on the host operating system)
- Everything else, we recommend installing in your Linux environment for ease of installation and ease of use. Although none of the software we give you is malicious, it's never a good idea to run untrusted programs on your host operating system.