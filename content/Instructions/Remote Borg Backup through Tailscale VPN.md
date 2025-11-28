---
title: Remote Borg Backup through Tailscale VPN
date: 2023-08-24 22:24
aliases: 
  - access apps through tailscale
  - remote borg using vpn
draft: false
tags:
  - docker
  - borg
  - scrutiny
  - vpn
  - guide
  - ssh
  - tailscale
  - backup
  - nas
  - repository
  - server
  - instance
  - openssh
  - asustor
  - lvm
  - linux
  - unix
  - ubuntu
  - acls
  - portainer
  - borgmatic
  - krusader
  - uptime
  - kuma
  - network
  - access
  - SMART
  - remote
  - unraid
  - guide
  - instruction
---
 
Let's hope you've never experienced the regret from wishing you'd kept a backup of your valuable data. Be it family photos, projects you've spent more time on than you'd like to admit or game save files with vivid memories attached, a good and proper backup should be a first priority for everyone. That (and the fact I've lost terabytes of data over the years) is why I am now setting up an offsite backup system for my Unraid NAS.

> ****THE 3-2-1 BACKUP RULE****  
> 3 different backups on  
> 2 different medias (i.e. magnetic tape and HDD /SSD) and  
> 1 of them offsite

Below are all the steps documented and explained as best as I can, so you can follow along, if you wish, to keep an up to date backup of your data secured in a offsite location.

>[!info] Prerequisites
>This tutorial expects you to already Docker running on your client (from here on out referred to as "main-server"). I'll also expect you not to have an OS on the machine that will host your backups using Borg repositories (referred to as "backup-server" below).

## Setting up the Main Server

I am going to use an Asustor AS6404T as my offsite backup-server, since its compact, quite power efficient... and since it's what I have laying around. It's populated with 4 chucked Buffalo NAS 2Tb drives, for a total of 8Tbs worth of capacity. Since my main NAS "only" has 8Tb of capacity, this is perfect for now.

Now, since the Asustor runs its own OS, we have to enable booting from USB on it. Refer to [this guide](https://github.com/mafredri/asustor_as-6xxt/blob/master/boot.md) for help on doing that.

Next, lets choose a OS and get it installed. I'll be running Ubuntu Server, since I'm used to it. But we'll be installing it a little differently. It must be installed on a USB stick as a fully bootable and persistent OS install.

#### OS Installation on USB stick

1. Download the Ubuntu Server iso from [ubuntu.com](https://ubuntu.com/download/server)
2. Flash it to a USB stick using, for example, [Rufus](https://rufus.ie/en/)
3. Grab a second empty USB stick, insert both into the NAS and boot up.
4. Run the OS installer and select the USB drive as the drive to install on. Some notable settings to do:  
    1. If your installing Ubuntu Server, in the menu with different add-on services, select __Docker__ and continue. It will install Docker for you automatically.  
    2. Install __OpenSSH server__. It can be turned off later if you don't want it running, but I like to always have a way to connect to my server via SSH.
5. Reboot when the installer is complete and boot up on your new OS

#### Setting up LVM and network shares

A quite good guide I followed step by step to set up my LVM disk share can be found [here](https://www.alibabacloud.com/help/en/ecs/use-cases/use-lvm-to-create-a-logical-volume) (also linked in the references). Since I don't want to just rewrite it whole all over again, I'm giving you the link and letting them do the talking.

We don't need to create a network accessible share since we'll only be accessing the data on our share from either Borg or a SFTP client like WinSCP.

#### Installing Docker on the Backup-Server

During the installation of Ubuntu Server, I chose to let it install Docker for me. If your OS doesn't give you that option during install, please refer to the excellent instructions on [Docker Docs](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository) regarding manual Docker install using their `apt` repository.

#### Installing Tailscale

Check out Tailscales own OS / distro-specific instructions for how to install it on your OS [here](https://tailscale.com/kb/installation/)

>[!tip]
>If you run  `tailscale up --advertise-exit-node` you can use your backup server as a VPN and route your traffic through it!

After you've installed Tailscale, just write `tailscale up` in the terminal. This will start Tailscale. The URL to authenticate the Tailscale client will appear in the terminal window, copy this to your browser and authenticate it.

Now, we got Tailscale running and logged in.

Next, go to the admin dashboard and under __Machines__, click the three dots related to the backup server and select __Disable Key Expiration__. If you don't, you will have to authenticate the backup server again after some time.

##### __Optional__

In the [admin dashboard]([https://login.tailscale.com/admin](https://login.tailscale.com/admin)) and go to __Access Controls__.  
Once there, define your host and access rules like this:

```json
"hosts": {
	"main-server": "<main-server-tailscale-ip-adress>",
	"backup-server": "<backup-server-tailscale-ip-adress>",
},
 "acls": [
	{
		"action": "accept",
		"src":    ["autogroup:member"],
		"dst":    ["main-server:*", "backup-server:*"],
    },
],
"ssh": [
	{
		"action": "check",
		"src":    ["autogroup:member"],
		"dst":    ["autogroup:self"],
		"users":  ["autogroup:nonroot", "root", "autogroup:member"],
    },
],
```

This mess of code means:

- `hosts` Defines the __Names__ and __IP-addresses__ of the individual machines
- `acls` Sets the rules of access for the different machines. The "*" means that any port the host listens on will be open for communications on the Tailnet  
    (the Virtual Private Network that Tailscale creates to enable your machines to communicate with each other is called a Tailnet)
- `ssh` As the name suggests, it defines which users and hosts that can ssh with each other.

Please refer to [Tailscales docs](https://tailscale.com/kb/1018/acls/) for further information on ACLs and managing a Tailnet

#### Setting up Portainer
>[!info] 
>I've transitioned away from using Portainer since Tailscale threw a real hissy-fit over Portainers use of port 9443 (the communications port for Edge clients). And also since Portainer is quite cumbersome to work with.

Before doing anything, I would recommend signing up for the free business license that will allow us three separate environments. This can be done on their signup page [here](https://www.portainer.io/take-3)

With that done, lets create the backup servers Portianer Edge Agent.

To do this, we need to go to the client and prepare a command to spin up a Portainer container.

1. Log in to the main servers Portainer Web UI `http://ip.of.client:9000`
2. Go to __Environments__ -> __+ Add Environment__ and select __Docker Standalone__.
3. Select __Edge Agent Standard__
4. Input the Tailscale IP of the main server. (DO NOT use the __host__ ACLs name we added in Tailscale earlier, it breaks Portainers connection for some reason. I may troubleshoot this at a later date.). Do this for both the __API Server URL__ and __Tunnel Server URL.__
5. Click __Create__ and scroll down to the Docker compose code. Copy this to the backup server and run.
6. It should give back a random string of numbers and letters. These identify the container in Docker. And for us, it tells us that it worked. The remote Portainer agent should now be connected.

#### The Borg Server

You'll find my modified Docker compose for the Server here: [[Borg Server -  Compose]]

Create a new __Stack__ in the backup servers Portainer and load up the compose code linked above. Modify it as necessary.

In the directory "/path/to/backup/dir/__sshkeys__", we are going to create a subdirectory named __clients__. In /sshkeys/clients, we are going to create as many subdirectories as we are going to have clients. You can always add to this in the future as your number of clients grows.  
In every clients subdirectory we will add their ssh.pub keys. Lets create them.

---

## Preparing The Clients

In Portainer, create a new stack and run the code for the [[Borg Client - Compose]]

As you probably noticed, I've specified a volume path for "/borg/backup" (the destination directory). In the final deployment, we'll want the backup directory to point to our remote Borg Server. But to be able to generate a SSH key pair and to authenticate with the server, we must have the container in a running state for some time (this container restarts every time a backup fails, and since it can't connect to the server yet, it will fail indefinitely if we don't point it to a temp directory to keep it alive while we prepare it's SSH communication. So lets get to it.

1. Make sure the borg client container has a path mapped to the /borg/backup volume.
2. Enter the containers terminal
3. Copy this command: `ssh-keygen -t rsa`
4. Enter a identifiable name for the key, ending with "_rsa"
5. Run the command again, but replace __rsa__ with __ed25519__ (they are different algorithms for encryption)
6. Copy the created files to the /clients/<client-name> directory on the server  
    Check our my post on how to set up [[Instructions for Docker SSH Public Key Authentication | SSH Public Key Authentication in Docker Containers]] to find out which permissions SSH clients wants for the public key directory and key files
7. Start the server and check the logs for `** Adding client <ssh-key-file-name>.pub with repo path /backup/<ssh-key-file-name>`
8. Run the initialization command for the Borg repo
    `borg init /path/to/repo -e repokey`
9. Back on the client, ssh into the borg server docker using `ssh <username>@<borg-server-IP>` . Accept the public key. Do this for both the LAN IP and for the Tailscale IP (you can find that IP under the __Machines__ tab on the Tailscale web admin dashboard).
10. Stop the container and change the __BORG_REPO__ from the local path to the ssh:// path, make sure to use the Tailscale IP to the server! Uncomment out the volume mapping to the local backup directory, since we don't need it no more. Feel free to delete it to save space.
11. The last thing to do is to add subdirectory mappings to the `/borg/data` volume. These subdirectories will allow us to map all our folders / shares we want Borg to backup. For example, you can add "/media/images" as "/borg/data/__images"__ and "/media/videos" as "/borg/data/__videos"__ so Borg doesn't mash them together when it backs them up.

Now we should have the repo all set up and ready to go! Check the logs for the client container and you should find that it says `Creating archive at ...` . If it does, that means it's working!

---

## A note about Borgmatic

I've now written a [[Configure Borgmatic & Recover Files | guide on how to set up Borgmatic]] (since I switched to using it because it can pause / unpause my Docker containers while backing up my appdata directory) Borgmatic is essentially Borg with a little different command / config structure.

---

## Fun Extras

By this point we should have a working Borg backup pipeline going through a VPN. Brilliant!  
Now, I atleast like to add some monitoring to my server. Some uptime monitoring and failure reporting. These containers are running either on my backup server or pinging it.

- [Scrutiny](https://github.com/AnalogJ/scrutiny) - a S.M.A.R.T (disk health) monitor with a Web UI and notification capabilities to, among other, Discord.
- [Krusader](https://hub.docker.com/r/binhex/arch-krusader) - a great file explorer that every server admin should have
- [Uptime-Kuma](https://github.com/louislam/uptime-kuma) - a uptime monitor with ability to send messages and warnings to, among many services, Discord

---

### Links to Useful Articles

- Enable USB Boot on Asustor NAS  
    [https://github.com/mafredri/asustor_as-6xxt/blob/master/boot.md](https://github.com/mafredri/asustor_as-6xxt/blob/master/boot.md)
- Tailscale Installation Instructions  
    [https://tailscale.com/kb/installation/](https://tailscale.com/kb/installation/)
- Setting up LVM  
    [https://www.alibabacloud.com/help/en/ecs/use-cases/use-lvm-to-create-a-logical-volume](https://www.alibabacloud.com/help/en/ecs/use-cases/use-lvm-to-create-a-logical-volume)
- Tailscale Network Access Controls  
    [https://tailscale.com/kb/1018/acls/](https://tailscale.com/kb/1018/acls/)
- Borg Documentation  
    [https://borgbackup.readthedocs.io/en/stable/index.html](https://borgbackup.readthedocs.io/en/stable/index.html)
- How to self-host a hardened Borg server - Sun Knudsen  
	[https://sunknudsen.com/privacy-guides/how-to-self-host-hardened-borg-server  
    ](https://sunknudsen.com/privacy-guides/how-to-self-host-hardened-borg-server)
- Borg Backup - Detailed Guide  
	[https://blog.unixhost.pro/2023/01/borg-backup-detailed-guide/](https://blog.unixhost.pro/2023/01/borg-backup-detailed-guide/)