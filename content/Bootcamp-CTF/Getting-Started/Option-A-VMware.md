---
title: "Option A: VMware Player setup"
description: setting up kali in vmware player
published: true
date: 2021-01-01T21:00:54.437Z
tags: 
editor: markdown
dateCreated: 2020-12-31T02:43:50.541Z
---

# Option A: VMware Player
VMware Player is one of the two most popular desktop virtualisation applications.  

# Download and install VMware Player
- VMware player can be downloaded and installed easily on Windows 10.  Pretty much any processor made after 2011 can support it.

- VMware Player 16 is free for personal use. The free version can not save snapshots, but that is fine for most things. Just select that you are using it non-commercially in the licence window that comes up the first time you run it after installation.



[Download VMware Player](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html)
	
After installing the virtualization software the next step is to install the operating system you want to virtualize, namely kali linux.

# Installing the guest operating system kali
While you can download the kali ISO file and install it into VMware just like you would normally install an operating system, the more efficient thing to do is to download a VMware image of the OS.  This image can just be imported into VMware and spun up in seconds, taking alot of the chore out of the process.  

[Kali VMware image Download](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/)

- Download the 32-bit image and unzip it into the location you want the virtual machine to live.  I put mine in my documents folder for reference. 
- Then open VMware and select "*open a virtual machine*" from the right side of the main window. 
- During the setup process select that you copied the image. 
- The default values for RAM and storage should be fine, but more ram does always make things smoother.  

After the image is installed you can select or right click your machine to start it.  The default username is kali and password is also kali.

- When starting the instance for the first time accept the prompt to install VM tools.  This will help with things like keyboard and mouse integration. 

# some final tips

- It is simple to import and export files. Just drag and drop in and out of the running machine window.
- The images may have pre-generated SSH keys, so it is good to make new ones or just not use it for things you care about.
- It would be smart to change the default password by running `passwd` in a terminal. 
