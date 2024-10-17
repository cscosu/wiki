---
title: "Option B: WSL"
description: 
published: true
date: 2021-01-14T22:22:41.063Z
tags: 
editor: markdown
dateCreated: 2020-12-29T18:00:50.246Z
---

# WSL (Windows Subsystem for Linux) Environment Setup
Windows subsystem for Linux is really a cool technology.  I use it for the majoity of my day to day activites just because it is so easy to pop in and out of. It also integrates with VS CODE making alot of things easier. 
WSL2 is a large step up from WSL if you have had past experence with it.  WSL2 can really be thought of a really low overhead VM running a linux kernal. 

One current limitarion of it though is that the network access is NATed from windows.   This means reverse shells and simple servers just dont work without playing with forwarding in windows. 

###  Installation

That said, lets look at setting it up.  I have both ubuntu 20.04 and kali in WSL. The install proccess for each is identical. 

- WSL2 requires for x64 systems: Version 1903 or higher, with Build 18362 or higher. 
- For ARM64 systems: Version 2004 or higher, with Build 19041 or higher.
But bottom line, builds lower than 18362 do not support WSL 2. 

First we need to enable the windows WSL feature. 
- Open a powershell terminal as administrator and run
`dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`

Next we need to enable windows virtulization.
- In a powershell terminal with administratior permisions run
`dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart`

And now restart your machine. 

Next we are going to update the linux kernal package.  Download and install the following [WSL2 Linux kernel update package for x64 machines](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

Next we need to set WSL2 as the default before downloading linux distro. 
``` wsl --set-default-version 2 ```

Now we are going to actually download and install the distro.  
Select your favorite from the list here [microsoft store](https://aka.ms/wslstore). 
After downloading and installing, WSL2 will either open a terminal window or the first time you start WSL2 it will prompt for a username and password. 

You can start WSL a few different ways. You can search for your distro name in the start menu, you can open a cmd windows and type either `wsl` or `bash` or the name of your distro like `ubuntu` or `kali`.  The same works from a powershell window as well.  

And that is pretty much it!  You can update and install and ssh and everything just like in a normal linux kernal.  Have fun! 
