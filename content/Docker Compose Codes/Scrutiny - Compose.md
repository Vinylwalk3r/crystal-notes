---
title: Scrutiny - Compose
date: 2023-11-02 21:47
aliases:
  - example
draft: false
tags:
  - docker
  - compose codes
  - scrutiny
  - disk
  - health
  - monitoring
  - alerting
---
 Scrutiny is a great S.M.A.R.T checker. It both logs the temps and status of all your drives and can also notify to a number of different services if it detects drive degradation.

```yaml
version: '3.5'

services:
  scrutiny:
    container_name: scrutiny
    image: ghcr.io/analogj/scrutiny:master-omnibus
    network_mode: bridge
    cap_add:
      - SYS_RAWIO
    ports:
      - "8250:8080" # webapp
      - "8086:8086" # influxDB admin
    volumes:
      - /run/udev:/run/udev:ro
      - /media/appdata/scrutiny/config:/opt/scrutiny/config
      - /media/appdata/scrutiny/influxdb:/opt/scrutiny/influxdb
    devices:
      - "/dev/sda"
      - "/dev/sdb"
      - "/dev/sdc"
      - "/dev/sdd"
      - "/dev/sde"
      - "/dev/sdf"
```

## Container Parameters

`container_name` The name of the container.

`network_mode` Which docker network to connect the container to. Among others can "host" or "bridge" be used.

`cap_add` This works like a very restrictive version of `--privileged`. Privileged gives the container ALL capabilities, whilst `cap_add` ONLY gives the specified capabilities to the container. For more info, refer to this [Docker Docs](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities).

### Ports

`8250` Port for the Web UI

`8086` Connect to this port with a app as __Adminer__ to manage the influx database.

### Volumes

`/run/udev` "ro" stands for "Read Only". udev is necessary for Scrutiny to be able to detect your drives and read S.M.A.R.T info from them

`/opt/scrutiny/config` Dir for all config files and other necessities for Scrutiny to run

`/opt/scrutiny/influxdb` A local database Scrutiny saves S.M.A.R.T info into.

### Devices

List all your disks here as detected by `smartctl --scan` when run on your host system. Check Scrutinys Github (linked below) for detailed instructions on what to do in a RAID config.

---

## Notifications

Open `scrutiny.yaml` in the "/config" directory. Scroll down to the "notify" section and configure as necessary. I only chose Discord and for that, I needed two things. A ****Webhook**** and a ****Token****.

The ****Webhook**** is created by:

1. Creating a Server (if you dont already have one)
2. Creating a text channel for your notifications (maybe named something like "name-of-computer scrutiny notifs")
3. Right clicking on the text channel and selecting __Edit Channel__. Then __Integrations -> Webhooks__ and creating a new one.
4. Copy the "Webhook URL" paste into a text editor. Your URL is going to look something like this:  
    [`https://discord.com/api/webhooks/5843905893058345903/jflkFJKLFJklfjdklsf7890f7fs95t87564F566dsf789sfwejwklföjdklsöfudasdF`](https://discord.com/api/webhooks/5843905893058345903/jflkFJKLFJklfjdklsf7890f7fs95t87564F566dsf789sfwejwklföjdklsöfudasdF) (Dont worry, this one is fake)
5. The first `5843905893058345903` is your ****Token**** and the longer `jflkFJKLFJklfjdklsf7890f7fs95t87564F566dsf789sfwejwklföjdklsöfudasdF` is the ****Webhook****. Paste these into the `discord://` in the scrutiny.yaml file. The ****Webhook**** goes first, then the ****Token****.
6. Uncomment the line you've edited in the scrutiny.yaml file. Save and start up the container.
7. Test your notifications by issuing this command in the containers terminal:  
    `curl -X POST` [`http://localhost:8080/api/health/notify`](http://localhost:8080/api/health/notify)

---

### Refrences

- Scrutiny by AnalogJ on Github  
	[https://github.com/AnalogJ/scrutiny](https://github.com/AnalogJ/scrutiny)