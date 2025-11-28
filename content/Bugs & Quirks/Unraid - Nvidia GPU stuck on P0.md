---
title: Unraid - Nvidia GPU stuck on P0
date: 2023-11-03 11:50
aliases: 
  - nvidia gpu powerstate stuck
draft: false
tags:
  - nvidia
  - gpu
  - powerstate
  - p0
  - p8
  - sleep
---
 
My Nvidia Quadro P400 gpu had this weird issue in my Unraid box, that it was stuck on P0 (P-States are power stages that the GPU uses to save power when idle. P0 is full power, whilst P8 is all idle.)

The fix I found is the command listed below. For me, it is a permanent fix persistent between reboots.

Input this command in Unraids terminal (accessible from the GUI).

`nvidia --persistenced` ​

Check that it worked by using this command and check the P value:

`nvidia-smi` ​

### Refrences

- __Nvidia P-State command__ by Ich777 on Unraid Forums  
	https://forums.unraid.net/topic/109548-solved-nvidia-p2000-power-state-idle/?do=findComment&comment=1231522