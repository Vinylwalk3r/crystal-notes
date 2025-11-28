---
title: Borg Server - Compose
date: 2023-08-18 21:34
aliases:
  - borg repository server
  - borg remote server
draft: false
tags:
  - docker
  - compose codes
  - borg
  - server
  - backup
  - ssh
  - repository
---
 
## Docker Compose

```yaml
version: '3'
services:
 borgserver:
  container_name: borg-server
  image: nold360/borgserver
  network_mode: bridge
  
  volumes:
   - /media/backup:/backup
   - /media/backup/sshkeys:/sshkeys

  ports:
   - "222:22"

  environment:
  # Additional Arguments, see https://borgbackup.readthedocs.io/en/stable/usage/serve.html
   BORG_SERVE_ARGS: ""

   # If set to "yes", only the BORG_ADMIN
   # can delete/prune the other clients archives/repos
   BORG_APPEND_ONLY: "no"

   # Filename of Admins SSH-Key; has full access to all repos
   BORG_ADMIN: ""
  restart: unless-stopped
```

#### | [GITHUB](https://github.com/Nold360/docker-borgserver) | [DOCKER HUB](https://hub.docker.com/r/nold360/borgserver) |

---

## ****Container Parameters****

`container_name` The name of the container.

`network_mode` Which docker network to connect the container to. Among others can "host" or "bridge" be used.

### ****Volumes****

`/backup` The location of the repositories. Borg will create sub-directories here which corresponds to the names of the ssh public key files in the `/home/borg/.ssh/` directory.

`/home/borg/.ssh/` The directory to store the .pub key files. IMPORTANT! Create sub-folders for each client. For example, for user 0 and 1, we create `/home/borg/.ssh/0` and `../.ssh/1` respectively and paste their keys in their respective folders.

### ****Ports****

`222:22` Since port 22 is occupied by the host OS (ssh), we remap it to 222.

### ****Environment Variables****

`BORG_SERVER_ARGS` If you need to specify any special Borg arguments it should use at launch, put them here.

`BORG_APPEND_ONLY` Setting this to YES allows clients to __ONLY__ add files to their repos. If you set this to yes, you'll have to specify a `BORG_ADMIN` which will be able to administer the repos.

`BORG_ADMIN` This specifies which user have full access to the repos. Points to a .pub key which identifies a SSH user. ONLY GRANT A TRUSTED USER THIS PRIVLIGE!

### ****Extras****

`restart` Is the restart policy for docker. Possible variables: "no", "on-failure[:max retries]", "always" and "unless-stopped". More info can be found [here](https://docs.docker.com/config/containers/start-containers-automatically/)

---

## ****Refrences****

- BorgServer by Nold360 - Github  
    [https://github.com/Nold360/docker-borgserver](https://github.com/Nold360/docker-borgserver)
- SSH Public Key - No supported authentication methods available (server sent public key) - AskUbuntu  
    [https://askubuntu.com/questions/204400/ssh-public-key-no-supported-authentication-methods-available-server-sent-publ](https://askubuntu.com/questions/204400/ssh-public-key-no-supported-authentication-methods-available-server-sent-publ)
- SSH keys - ArchiWiki  
    [https://wiki.archlinux.org/title/SSH_keys](https://wiki.archlinux.org/title/SSH_keys)
- How to Setup SSH Passwordless Login in Linux - TechMint  
    [https://www.tecmint.com/ssh-passwordless-login-using-ssh-keygen-in-5-easy-steps/](https://www.tecmint.com/ssh-passwordless-login-using-ssh-keygen-in-5-easy-steps/)
- Start Containers automatically - Docker Docs  
    [https://docs.docker.com/config/containers/start-containers-automatically/](https://docs.docker.com/config/containers/start-containers-automatically/)