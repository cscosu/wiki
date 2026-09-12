---
title: Recommended setup
description: Introductory setup to get you started
published: true
date: 2026-09-11T01:49:34.291Z
tags: intro, begin, howto, how-to, new, start, hack, hacking
editor: markdown
dateCreated: 2026-09-11T01:49:34.291Z 
---
# Linux
The three main distro's I recommend purely for the convenience of shipping all the packages you want are kali, parrot, or arch.

## Virtualization
You can get linux without running on bare metal. This requires the least amount of investment as you can still run your operating system of choice and learn how linux works.

### WSL
If you are on a windows you can either install a hypervisor or run `wsl --install kali-linux` in your windows terminal.

Then setup your user and password. You will then be popped into a kali linux shell.

If wsl is already installed and you don't get a kali shell on subsequent runs of `wsl`
you can use `wsl -d kali-linux` to run specifically the kali-linux machine you installed 


### Setup virtual machine
Hypervisors are softwares that manage virtual machines. There are many, use whichever you like. If you don't know what you like use virtualbox. It is decently beginner friendly and supported.

If you are on an arm system you will likely only want to mess with parrot or kali in your virtual machine as those have better arm support than arch.

Go to the website of your chosen distro and download the iso that matches your architecture. I would recommend checking the hash that the website stores and the hash you get from the downloaded iso to make sure they match

If the iso is an archive, be sure to right click and extract it.

Once you have done all that create a new virtual machine in virtualbox and select the iso you downloaded as well as the amount of storage, ram and cpu you want. These can all be changed post install. 

## dual booting
Dual booting is primarily a windows opiton. It is possible to do on a mac but is a bit more difficult. Dual booting quite nice if you want linux but don't want to jump into the deep-end with no way back to shore.


It is generally the best and least disruptive if you have two storage drives in your computer and run linux on one and windows on the other. However, if you need to use the same drive you have two options. Firstly, you can mannually partition, which teaches you about creating and destroying partitions. Or you can use one of the common linux installers fedora, ubuntu, debian who can do this partitioning automatically for you.

Should you chose to mannually partition, you can open up the windows disk manager and shrink the drive you are running down. Most installers will do this step for you, and it can be risky but you would miss the wonderful learning experience of creating and destroying partitions. 

These "hacking" images really only need 25-50 Gb of storage without swap. I have found it best to leave about a Gb of space between windows and linux partitions. You can accomplish this by adding a 1 Gb partition right after the windows partition, installing linux then deleting that partition.

You can follow the rest on [this guide](https://linuxblog.io/dual-boot-linux-windows-install-guide/)

## FAQ
### Do I need this exact software?
No cybersecurity is a very creative art. While the setups outlined above are what I have found to be the easiest for me, that doesn't mean you need to use them. This guide is more for a setup if you have no clue where to get started

### Is open source actually more secure
It depends. I presonally think so and would rather trust an open source project than trust a big tech coorporation. But your definition of secure might be different and trust different entities. 
