---
title: Activate & Deblaot Windows 10 / 11
date: 2024-03-15 23:17
aliases: 
  - Free Windows Activation
  - Stop Windows Spying
draft: false
tags:
  - windows
  - debloat
  - lite
  - activate
  - license
  - free
---
 
I hate having to reinstall Windows. It's so much more tedious and crap-filled than it has to be. Microsoft constantly pesters us with "Do you allow us to collect this data?" "Can we have that data?" "GIVE US YOUR DATA!". UUHH. So that's why we're here today, to shut Microsoft up a bit. This works on both Win 11 and 10.

---

First some context. I'll write this in the sequence I did it during my latest reinstall. Follow them in your own order, if you don't wish to follow mine.

## Activating

NEVER have I paid for this OS. Probably not many of you either. So lets activate.

1. We begin by callign upon the "Software Licensing Management Tool" vbs script to replace the product key in your Windows installation. Do this by opening up cmd by WIN+R and paste this command:  
    `slmgr /ipk W269N-WFGWX-YVC9B-4J6C9-T83GX /AcceptEula`
2. Next, we will change which server the KMS service will call to check the product keys legitimacy.  
    `slmgr /skms kms8.msguides.com`
3. And lastly, lets check our key and activate our Windows install.  
    `slmgr /ato`

---

## Debloating

If your ever installed Windows, you will most probably have been frustrated with the amount of bloat and sheer useless crap Microsoft delivers Windows with. So lets get cleaning.

### Remove MS Store Apps Utility

This littls number by Digressive is hosted on Github and is a script that, when paired with a simple text file, removes almost any preinstalled software in Windows.  
It uses the `Remove-AppxPackage` with the "-package" flag to remove the apps.

But before we can use it, we have to prepare Powershell beforehand.

##### Preparing Powershell

We have to change the Execution-Policy so Powershell has permission to load custom scripts. Do this by:

1. Open Powershell
2. Run `Set-ExecutionPolicy`
3. Type in "Unrestricted"

Done.  
****DONT FORGET TO DO THIS AGAIN AND SET ExecutionPolicy to "DEFAULT"**** _****AFTER****_ ****USING THE UTILITY****!!!

##### Running the Utility

1. Open Powershell as Admin
2. Install the utility by running `Install-Script -Name Remove-MS-Store-Apps`
3. Next, Run the utility using `Remove-MS-Store-Apps.ps1 -PCApps` This will generate a list with all the apps that you have installed. Either copy the ones you want into a text file (which I recommend placing in "C:\" for easy access later) or just copy the ones I removed:

```cmd
Clipchamp.Clipchamp
Microsoft.549981C3F5F10
Microsoft.BingNewsMicrosoft.BingWeather
Microsoft.GamingAppMicrosoft.GetHelpMicrosoft.Getstarted
Microsoft.MicrosoftEdge.StableMicrosoft.MicrosoftOfficeHubMicrosoft.MicrosoftSolitaireCollection
Microsoft.Paint
Microsoft.People
Microsoft.PowerAutomateDesktop
Microsoft.SecHealthUI
Microsoft.StorePurchaseApp
Microsoft.Todos
Microsoft.WindowsAlarms
Microsoft.WindowsCamera
microsoft.windowscommunicationsapps
Microsoft.WindowsFeedbackHub
Microsoft.WindowsMaps
Microsoft.Xbox.TCUI
Microsoft.XboxGameOverlay
Microsoft.XboxGamingOverlay
Microsoft.XboxIdentityProvider
Microsoft.XboxSpeechToTextOverlay
Microsoft.YourPhone
Microsoft.ZuneMusic
Microsoft.ZuneVideo
MicrosoftCorporationII.QuickAssist
MicrosoftTeams
```

4. Next, remove your selected apps using `Remove-MS-Store-Apps.ps1 -List C:\{your_text_file_here}.txt`
5. Done

### WinUtil

This last piece of software is my new favourite. It help you quickly install and remove apps and can modify Windows settings from one easy-to-access place!

1. Open up a Powershell window in Admin
2. Input `iwr -useb` [`https://christitus.com/win`](https://christitus.com/win) `| iex`
3. In the __Install__ tab you can choose which apps you want to install or remove. A usefull button is "Get Installed". It checks which apps you have installed and ticks those boxes.
4. __Tweaks__ is super usefull. I run most of these, but choose the ones you want. DISABLE "MOUSE ACCELERATION" THOUGH! and "Bing search in Start Menu"
5. One thing I REALLY like in __Features__ is the easy access to the "Set Up Autologin" which I always have on my desktop.

Now, do a reboot to set everything up and your good to go!

### WinSetView

One more little program I want to highlight is WinSetView. The default "Group by..." setting in Explorer bugs me everytime it's enabled. I REALLY dislike it. But disabling it universaly is REALLY tedious. Thats where this little app comes in. And it's Portable to!

One nice feature it has is that when you set, say, Pictures to "Large Icons", then all Picture folders below inherits it, so Pictures in Onedrive and Search results all follow what you set for the Pictures folder.

Also don't forget to go into the "Options" menu and config it how you want it to be. I don't like Compact mode for example. You can disable it there, and Unhide Appdata to. Nifty!

---

#### Refrences

- Software Licensing Management Tool on SS64.com  
	[https://ss64.com/nt/slmgr.html](https://ss64.com/nt/slmgr.html)
- About Execution Policies on Learn Microsoft  
	[https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.4](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.4)
- Remove-MS-Store-Apps-Utility by Digressive on Github  
	[https://github.com/ChrisTitusTech/winutil](https://github.com/ChrisTitusTech/winutil)
- WinUtil by ChrisTitusTech on Github  
	[https://github.com/ChrisTitusTech/winutil](https://github.com/ChrisTitusTech/winutil)
- WinSetView by Les Fertch on Github  
	[https://github.com/LesFerch/WinSetView](https://github.com/LesFerch/WinSetView)