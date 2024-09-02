---
title: Configuring the AIO Downloader Stack
date: 2023-11-03 23:41
aliases: 
draft: false
tags:
  - docker
  - instruction
  - gluetunvpn
  - discord
  - deluge
  - bot
  - lidarr
  - readarr
  - sonarr
  - radarr
  - notifications
  - qbittorrent
  - deluge
  - transmission
  - vpn
---
This guide will show you how to set up and configure a docker stack that will download all sorts of media through a vpn and be able to take requests from a Discord bot we will set up.

You can find the compose codes necessary for setting this up yourself below

[[AIO Downloader Stack - Compose]]

---

Just a quick little something before we begin. I'd advise setting your folder structure for your *arr apps like:

/data/media/<subfolders>  
/data/torrents/<subfolder>  
/data/usenet/<subfolders>

This is so that we can utilise __Hardlinking__ (read more about it below) and limit the amounts of read/writes on our disks. And it makes it all clean and simple to look through as well.

---

## Building the Stack

We'll want to choose what kind of media we will be downloading and build our stack from there. Or just go all out and add everything to your stack.

My modified compose codes are available [[AIO Downloader Stack - Compose]].

Copy and paste the codes for all the containers your going to want to run into one compose code and change the necessary path variables.

The tricky ones are the OpenVPN and Wireguard settings for GluetunVPN. Since qdm12 has done such a awesome job of explaining the process of configuring Gluetun for different VPN providers, I'll just be linking to his wiki [HERE](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers). Please refer to it for guidance on setting up Gluetun with your VPN of choice. I'm using Mullvard.

Next, spin up the stack. This is done a little differently in [Portainer](https://docs.portainer.io/user/docker/stacks/add), Unraids [Docker Compose plugin](https://docs.ibracorp.io/docker-compose/docker-compose-for-unraid) or docker run commands. Please refer to the relevant documentation for how to deploy Docker Compose codes in your environment.

---

## Configuring the Apps

#### Adding indexers to Prowlarr

In the ****Indexers**** tab, then click the Plus icon that says "Add indexer". If you don't have accounts on any special indexers, just select "Public" in the __Privacy__ menu. Change the indexer settings as necessary and save. Repeat for as many indexers as you want.

#### Getting API keys from apps

This works the same for all the *arr apps.  
Go into the Web UI of the application. Next, go into __Settings__ -> __General__ -> scroll down to __Security__. There, copy the ****API key****.

#### Adding Syncs to Prowlarr

Get the _****API key****_ from the app you want to add to Prowlarr.  
Go to ****Settings**** -> ****Apps**** and click the big plus icon under "Applications".  
Set your __Sync Level__ to "Full Sync"(this will allow Prowlarr to both add and remove indexers from the app. Useful for centralizing index control.)  
In the two __... Server__ fields, we will put [`http://127.0.0.1:`](http://127.0.0.1:9696) and then the port of the application we want to point to. Select the __Sync Category__ as necessary.

##### Ports:

- Prowlarr - 9696
- Sonarr - 8989
- Radarr - 7878
- Readarr - 8787
- Lidarr - 8686
- Bazarr - 6767

---

## General App Settings

### Adding Downloaders

This works the same for all the *arr apps.  
Go to __Settings__ -> __Download Clients__ and click on the plus button. In the __Host__ field, put [`http://127.0.0.1`](http://127.0.0.1) since we're accessing it from behind the VPN. Add the __Port__:

- Deluge - 8112
- QBittorrent - 8180
- Transmission - 9091

Add the password for the login to the download clients Web UI as well. In the __Category__ field, I recommend putting the name of the app that is making the request (Prowlarr, Sonarr, etc). If your using QBittorrent, we're going to be able to use that Category later.

Now, go back to the __Indexers__ tab and click the __Sync App Indexers__ button. The indexers you've set up in Prowlarr will now be synced to all your *arr apps!

### Media Management Settings

Use this with Sonarr and Radarr. If you don't find this setting, click __Show Advanced__ in the top bar!

Tick the box __Use hardlinking instead of copy__. This will reduce the wear on your drives, since Sonarr wont copy the file to the new directory. It will create a hardlink (think of it as a shortcut in Windows) to the file in the new directory, greatly reducing read/writes for the disk.

### Add Connects (Sonarr / Radarr)

- ****Discord****  
    `Name` Enter whatever name you want in the __Name__ field.  
    `Notification Triggers` Choose the triggers you want to trigger this notification. Mine are all between "On Grab" and "On Movie File Delete for Upgrade".  
    `Webhook URL` You get the __Webhook URL__ by creating a Webhook for a text channel in a Discord server. Right click on the channel and go to __Edit Channel__ -> __Integrations__ -> __Webhook__ -> __New Webhook__. Copy the ****Webhook URL**** and paste in this field.  
    `Username` The name you want the app to have in Discord  
    `Avatar` URL to a image you want to be the apps profile picture in Discord  
    I'd leave the rest of the fields blank unless you want to make specific customizations.

I have not added further Connections, so I can not advise more on how to set them up.

### Other Tips & Tricks

****Light / Dark Modes****  
Light / Dark mode can be set in __Settings__ -> __UI__ -> __Style__ -> ****Theme****  
This only works for Radarr though. Sonarr doesn't have selectable themes (as of writing) and the other *arrs have dark mode by default.

****Dates & Times****  
Set the dates in __Settings__ -> __UI__ ->****Dates**** and the calendar just a few rows up.

****Tasks****  
You can manually trigger different task by going to __System__ -> ****Tasks****

****Backups****  
Backups can be viewed, downloaded or restored from __System__ -> ****Backup****

****Logs****  
Some *arr containers allow you to view live logs by going to __System__ -> ****Events****

---

## Adding Discord Requests

Now, this is a fun one. We can deploy a Discord bot that allows us to make movie / tv requests directly from a Text channel and our stack will automatically search, download and organize it for us! Especially useful for shared media libraries.

First, your going to want to deploy the [[AIO Downloader Stack - Compose#Doplarr|Doplarr]] container using the compose codes.

Change the "localhost" in the `Sonarr_URL` and `Radarr_URL` to "127.0.0.1".  
The `API keys` will you know where to find by now. Scroll up to [[Configure a AIO Downloader Stack#Getting API keys from apps|Getting API keys from apps]] for instructions.  
`Discord Token` (Im stealing this straight from Kiranshilas great guide on it. Consider this a backup of their work)

> The first step in configuration is creating the bot in Discord itself.  
> 1. Create a new [Application](https://discord.com/developers/applications) in Discord  
> 2. Go to the Bot tab and add a new bot  
> 3. Copy out the token, this will be used for the DISCORD__TOKEN setting  
> 4. Go to OAuth2 and under "OAuth2 URL Generator", enable `applications.commands` and `bot`  
> 5. Copy the resulting URL and open it in a web browser to authorise the bot to join your server

Now it should work and be ready for all your media requesting needs!

---

### Refrences

- VPN providers config guides by qdm12 on GluetunVPN Wiki - [https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers)
- *arrs info on Servarr Wiki - [https://wiki.servarr.com](https://wiki.servarr.com)
- Doplarr configuration by Kiranshila on Github.io page - [https://kiranshila.github.io/Doplarr/#/configuration](https://kiranshila.github.io/Doplarr/#/configuration)