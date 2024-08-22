---
title: Borg Client - Compose
date: 2023-08-16 22:01
aliases:
  - borg docker
draft: false
tags:
  - docker
  - compose codes
  - borg
  - client
  - backup
  - ssh
  - repository
---
## Docker Compose

```yaml
version: '3.3'
services:
 borgbackup:
  container_name: borg-backup
  image: pschiffe/borg
  hostname: {hostname}-borg
  network_mode: {network-name} 
  
  volumes:
   - {path/to/appdata/}borg:/root
   - {path/to/source/files}:/borg/data:ro
   - {path/to/temporary/backup/dir}:/borg/backup

  ports:
   - "222:22"
  
  environment:
  # - BORG_REPO=ssh://borg@{adress.to.remote.repo}:222/./
  - BORG_REPO=/borg/backup
   - SSHFS_IDENTITY_FILE=/root/.ssh/id_ed25519
   #- SSHFS_GEN_IDENTITY_FILE=1
   - BACKUP_DIRS=/borg/data
   - BORG_PASSPHRASE=
   - COMPRESSION=lz4
   - PRUNE=1
  
  devices:
  - /dev/fuse
  
  cap_add:
  - SYS_ADMIN
  
  security_opt:
  - label:disable
  
  restart: unless-stopped
```

### | [GITHUB](https://github.com/pschiffe/docker-borg) | [DOCKER HUB](https://hub.docker.com/r/pschiffe/borg/) |

---

## ****Container Parameters****

`hostname` must be set to something different than the OS hostname. A easy solution is to set "-borg" at the end of the OS hostname

`network-mode` can be anything (host, bridge, custom network) as long as its reachable from the Tailscale network that the server is in.

### ****Volumes****

`/root` is the config file directory. Borg stores its files here

`/borg/data` is the directory containing the files you want backep up. THIS IS NOT THE BACKUP REPOSITORY ^vol-borg-data

### ****Ports****

`port 222:22` Since port 22 is often used by host OSes for ssh, we have to use a different port for the docker container ssh. In this case, 222.

### ****Environment Variables****

`borg_repo` The destination borg will send the files from `/borg/data` to.

- If it's a ****local repo**** just put in a path like this: `/path/to/destination/folder`.
- It it's a ****remote repo****, you must put ssh:// before the ssh adress. Important is that the directory structure at the remote repo shall not be put here. If the path to the repo on the server is "/path/to/client_repo_folder" we shall still just put /./ as the path in the ssh command. I guess this is because that the borg-server automaticly connects the client to the root of the clients repository directory. So "/" for the client corresponds to "/client_repo_folder" on the server side

#### ****SSH Keys****

`SSHFS_GEN_IDENTITY_FILE=1` If this is set to 1, SSH keys will be generated in the directory set by `SSHFS_IDENTITY_FILE` if they dont already exist. I left the line in the code if I ever need it, but because how we will generate SSH keys manually in the main article, I just commented it out.

`SSHFS_IDENTITY_FILE` This tells Borg where and which key.pub file to use for SSH identification. (We will generate these from within the container after we start it!)

`backup_dirs` Tells Borg which directories (in the container) it shall backup. Set this as the same as you set the data dir.

#### ****Backup Dir & Settings****

`borg_passphrase` Set this to the `repokey` password. Defaults to none. Only the `repokey` mode encryption is supported by this Docker image. [More info](https://borgbackup.readthedocs.io/en/stable/usage.html#borg-init)

`compression` Tells borg which compression to use. Defaults to lz4. [More info](https://borgbackup.readthedocs.io/en/stable/usage.html#borg-create)

`prune` if set, prune the repository after backup. Empty by default. [More info](https://borgbackup.readthedocs.io/en/stable/usage.html#borg-prune)

- ****PRUNE_PREFIX**** - filter data to prune by prefix of the archive. Empty by default - prune all data
- ****KEEP_DAILY**** - keep specified number of daily backups. Defaults to 7
- ****KEEP_WEEKLY**** - keep specified number of weekly backups. Defaults to 4
- ****KEEP_MONTHLY**** - keep specified number of monthly backups. Defaults to 6

#### ****Extras****

`devices` `cap_add` and `security_opt` are variables I am testing to see which are necessary and which arent. So far, `devices` and `cap_add` are necessary

---

### ****Refrences****

- Docker-Borg by Pschiffe - Github  
	[https://github.com/pschiffe/docker-borg](https://github.com/pschiffe/docker-borg)