---
title: Sync Obsidian notes using WebDAV
date: 2023-10-30 19:45
aliases: 
  - Obsidian Sync w. Webdav
  - WEbdav syncing Obsidian Vaults
draft: false
tags:
  - obsidian
  - webdav
  - server
  - domain
  - ssl
  - notes
  - syncing
  - docker
  - remotly
  - save
  - plugin
  - apache
  - instruction
---
 
> Why set up a manual sync when Obsidian offers integrated sync for just 8 to 10 bucks per month?

First of, even if most people probably wont hit it, there is a 10Gb upper limit for your vault size. If you sync a lot of images, it adds up fast.  
Second is security concerns. Data stored on someone else's server is not your data anymore. Whether that's a big deal or not, that's your decision. But here's how to set up Obsidian to sync using a dockerized WebDAV server.

---

## Step 1. Preparing a Domain

I'm suspecting you'll wanting to access your WebDAV server from anywhere. If not, just skip this step. But for all else, lets get into it.

Since I use Cloudflare for my DNS records, I set up a new CNAME record and point it to the main A record. You'll do this in the Cloudflare Dashboard -> DNS -> Records.

With that done, time for Nginx. I'm using NPM.

- Begin with getting a SSL Certificate. Do this by going to  
    __SSL Certs -> New Certificate__  
    and select DNS Challenge.  
    
- Proxy Host next. Click on "Add Proxy Host" and enter as follows:  
    `Domain Names` = "webdav.yourdomain.com"  
    `Scheme` = HTTP  
    `Forward Hosts` / IP = your hosts LAN IP, for example 192.168.1.13  
    `Forward Port` = 8385, or a custom port if you need to.  
    `Block Common Exploits` = Enable  
      
    Under the SSL tab:  
    `SSL Certificate` = the SSL Cert we got from the step above.  
    `Force SSL` = Enable  
    `HTTP/2 Support` = Enable

Now we have a domain, a SSL Certificate and a reverse proxy ready. Time to Docker!

## Step 2. Installing WebDAV

For this use, I've opted for [Apache-WebDAV](https://github.com/mgutt/docker-apachewebdav) (since it's what I first found in Unraids CA plugin). Lets get it installed.

These are the settings that work for me:  
`Extra Parameters` = --memory=1G  
`Network Type` = either __Bridged__ or a custom network (use docker network create newnetworkname)  
`WebDAV Share` = /path/to/persistent/storage (this is for the files the server will share!)  
`Webserver Port` = 8385  
`Domains` = webdav.yourdomain.com  
`Base URL` = /  
`Authentication` = Basic  
`Config` = /path/to/config/folder

Now, here is where we're going have to use a little trickery. If this WebDAV server only will serve one user, just put in your username and password in the `Username` and `Password` variables in the Unraid GUI / docker run command. But if you want multiple users (or just room for expansion) we'll need to make another step.  

Firstly, add a new __path__ and set it to "/path/to/config/webdav/user.passwd".  
We need this path mapped so we can touch it form inside the running container. This is because it will set the permissions right so the file will be accessible by the container.

Start the container now, I'll wait.

With the container started we're going to put our login details into the user.passwd file using these commands:

```
touch user.passwd
htpasswd -B user.passwd alice
htpasswd -B user.passwd bob
```

More information about multiple user can be found on its [Github repo](https://github.com/mgutt/docker-apachewebdav#authenticate-multiple-clients).

Now, you should have a working WebDAV server that is accessible from a external domain URL. If so, Congrats!

---

## Step 3. Setting up Obsidian

Inside Obsidian, go to Settings -> Community Plugins -> Browse and search for __Remotely Save__ and install it. Then Enable it. It should now pop up in the scrollable list to the left.

In Remotely Saves settings, these work for me:  
`Remote Service` = WebDAV  
`Server Adress` = https://webdav.yourdomain.com  
`Username` = your username from user.passwd or username variable if set  
`Password` = your password  
`Auth Type` = Basic  
`Change the Remote Base Directory` = I set mine to myname_obsidian, that way every users directory is separated by user and application.

You may set the rest of the plugins setting as you wish, these were the required once for getting the sync up and running. You may wish to set `Run Once on Starup Automatically` to something quick on start, say 1 second. Set it as you wish.

Now, if you try it, you should have a proper working Obsidian sync using a self hosted WebDAV server. Well done!

---

### ****Refrences****

- Apache-WebDAV by MGutt on Github  
	 [https://github.com/mgutt/docker-apachewebdav](https://github.com/mgutt/docker-apachewebdav)
- Obsidians Remotely Save Plugin on Github  
	[https://github.com/remotely-save/remotely-save](https://github.com/remotely-save/remotely-save)