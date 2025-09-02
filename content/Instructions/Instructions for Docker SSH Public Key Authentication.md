---
title: Instructions for Docker SSH Public Key Authentication
date: 2023-08-20 17:30
aliases: 
  - Public Key SSH Auth
  - SSH Auto Login
draft: false
tags:
  - ssh
  - docker
  - public
  - key
  - auth
  - authentication
  - client
  - remote
  - server
  - instruction
---
 
In this guide, I'll walk you through how to set up a SSH connection between a client and server Docker container on different hosts and enable Public Key Authentication between them.

### SSH Public Key Auth for Docker Containers

First we begin with setting up our clients Docker container. These steps are written after setting this up with a [[Borg Client - Compose]] container. It should be similar to any other container, but some things (like the ssh keys directory mapping) may differ.

##### Client

1. Enter the console of the client Docker container  
    (I am using Portainer so I just click the console button, otherwise use `docker exec -it {container-name} /bin/bash`)
2. Execute ssh-keygen and create both ed25519 and rsa key pairs  
    (use `ssh-keygen -t {algorithm name}` to set the specific algorithm to create with)
3. Move the .pub keys to the servers ssh path. (`/sshkeys/client/{client-folder}` for example)

Now we have generated a key pair that identifies our client Docker container and we've also moved the keys to the ssh key directory for the servers Docker container. Now lets set up the server

##### Server

This path is written for the [[Borg Server -  Compose]]. I'm using for my backups. But it should be mostly similar for any Docker container using SSH.

1. Enter the terminal and execute  
    `/chmod 700 /path/to/key/folder`
2. Then do these commands for every key file  
    `/chown {user-in-docker-container} {key.pub}`  
    `/chgrp {user-in-docker-container} {key.pub}`
3. Start the container when done

Alright! We should be all done on the server now. Lets finish up on the client and test our connection!

##### Client

1. Open the containers console
2. SSH into the Borg container on the Server
3. Accept the fingerprint key from the server
4. Restart both the client and server containers (Not always needed but it can't hurt)

And if we're lucky, that should be it. If it all works, now you should have a working SSH tunnel between your Docker containers authenticated using public key, so they don't have to log in using passwords.

---

### SSH Folder/File Permissions

SSH is very strict with who it wants to have access to the SSH key directory and the individual files in said directory. These are the permissions it wants to see.

Set the permissions for the ssh key directory to:

- Directory: `700`
- Public key file: `640`
- Private key file: `600`

For example, my folder structure is `./media/sshkeys/clients/{client-name}` and inside the folder are the public key files (.pub). So my permissions are 700 for {client-name} and 600 for my public keys. It seems ssh clients will rather have to strict permissions than to permissive.

---

__The following section is for the Host. It is NOT needed for enabling Pub Key Auth for Docker containers. Do this at your own risk.__

### SSHD_Conf Parameters

Modify `etc/ssh/sshd_conf` on the host OS and add  
`PubkeyAuthentication yes`  
`PasswordlessAuthentication yes`  
to the file.

Then reload the sshd service via:  
`systemctl restart ssh.service`