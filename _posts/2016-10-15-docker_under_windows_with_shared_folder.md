---
layout: post
title: Run docker under windows with or without shared folder
---

**Problematic:**
be able to launch docker from windows, edit my project files from a local folder, the files are updated on the docker instance.


**First attempt: **
 - share my local windows drive via Docker Settings dialog
 - mount my folder as a volume in my docker instance
 - result: the files are present on the docker instance but impossible to change the user/group of the files (stays root/root)
 - it seems that windows rights system interferes
 
** Second Attempt: **
 - let's use sshd to copy via sftp my files
 - but it's a very bad idea, as when I rebuild my docker instance, all the files will be lost and everything 
 will be needed to be copied again

** Third Attempt: **

use of Docker Toolbox
in this case, we will pass by a virtualbox image that can support shared folder(but we will not use it!) , but most of all that will  contain all of our source code and libraries.
As the docker machine abstract the underlying Operating system, we don't have anymore the problem of windows path size for nodes modules.

Note: before reading the next tutorial, don't install tyhe NDIS6 driver, if you have installed NDIS6 with VirtualBox and you have BSOD (Blue Screen Of the Death), try to use NDIS5 instead because there is a known issue with the new NDIS6 driver. Uninstall Virtualbox and try reinstalling the last version with a parameter

> VirtualBox-5.0.11-104101-Win.exe -msiparams NETWORKTYPE=NDIS5

Then follow the instructions of this tutorial: https://docs.docker.com/machine/drivers/hyper-v/

Launch the docker machine from a Powershell with elevated rights (administrator mode) and cross your fingers
> docker-machine create -d hyperv --hyperv-virtual-switch "Primary Virtual Switch" dev

Next step, is to to execute docker containers via docker-compose from the docker-machine you have just created

**Problematic2:**

nodejs modules are problematic under windows as there is a lot of sub directories, sometimes you can have a problem with 
path that is limited to 260 characters
