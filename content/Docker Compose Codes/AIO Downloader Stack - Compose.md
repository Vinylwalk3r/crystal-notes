---
title: AIO Downloader Stack - Compose
date: 2023-11-01 23:05
aliases:
  - arr stack
  - media downloader dockers
draft: false
tags:
  - docker
  - compose codes
  - downloader
  - gluetunvpn
  - vpn
  - arr
  - containers
  - starr
  - prowlarr
  - sonarr
  - radarr
  - readarr
  - bazarr
  - qbittorrent
  - deluge
  - transmission
  - torrent
  - discord
  - doplarr
  - bot
---
 The Docker compose codes below are a build-your-own-stack collection of already configured codes ready to be assembled to a stack or (with slight modifications) used independently.

A supplementary guide for how to configure these applications are available below:

[[insert link to guid here]]

I you want to use any of these codes without the VPN, just remove this and it will work:

```yaml
depends_on:
  GluetunVPN:
     condition: service_healthy
```

---

### [GluetunVPN](https://github.com/qdm12/gluetun)

The backbone of the stack. All the traffic from the other applications will tunnel through this one. The settings you want to edit are the ones in "{}" (OpenVPN and Wireguard).  
If you want to add other applications through the VPN, add ports to `ports` and `FIREWALL_INPUT_PORTS`. Then change the "network mode" in the new container to `container:GluetunVPN` .

```yaml
version: "3"
services:

  GluetunVPN:
    image: qmcgaw/gluetun:latest
    container_name: GluetunVPN
    network_mode: bridge
    # line above must be uncommented to allow external containers to connect. See https://github.com/qdm12/gluetun/wiki/Connect-a-container-to-gluetun#external-container-to-gluetun
    cap_add:
    - NET_ADMIN
    devices:
    - /dev/net/tun:/dev/net/tun
    ports:
    - 8310:8000/tcp # HTTP Server
    - 8888:8888/tcp # HTTP proxy
    - 8388:8388/tcp # Shadowsocks
    - 8388:8388/udp # Shadowsocks
    #- 9696:9696 # Prowlarr
    #- 8989:8989 # Sonarr
    #- 7878:7878 # Radarr
    #- 8686:8686 # Lidarr
    #- 8787:8787 # Readarr
    #- 6767:6767 # Bazarr
    #- 8112:8112 # Deluge
    #- 8180:8180 # QBittorrent
    #- 9091:9091 # Transmission
    volumes:
    - /path/to/appdata/gluetun:/gluetun
    environment:
    # See https://github.com/qdm12/gluetun/wiki
    - VPN_SERVICE_PROVIDER=mullvad
    - VPN_TYPE=openvpn
    # OpenVPN:
    - OPENVPN_USER={YOUR-USERNAME-HERE} 
    - OPENVPN_VERBOSITY=2
    - SERVER_COUNTRIES={COUNTRY-OF-VPN-SERVER} 
    - SERVER_CITIES={CITYNAME-OF-VPN-SERVER} 
     # Wireguard:
    - WIREGUARD_PRIVATE_KEY={YOUR-WIREGUARD-KEY-HERE} 
    - WIREGUARD_ADDRESSES={ADRESS-LIKE 10.10.10.10/32}
    # Timezone for accurate log times
    - TZ=Europe/Stockholm
    # Server list updater. See https://github.com/qdm12/gluetun/wiki/Updating-Servers#periodic-update
    - UPDATER_PERIOD=24h
    - UPDATER_VPN_SERVICE_PROVIDERS=
    #Remove the Firewall Input Ports that you wont use!
    - FIREWALL_INPUT_PORTS=9696,8989,7878,8686,8787,6767,8180,9091,8112 # Prowlarr, Sonarr, Radarr, Lidarr, Readarr, Bazarr, QBittorrent, Transmission, Deluge
    # DOT (DNS over TLS) for Outbound
    - DOT=on
    - DOT_PROVIDERS= cloudflare
    - DOT_PRIVATE_ADRESS= 127.0.0.1/8,10.0.0.0/8,172.16.0.0/12,169.254.0.0/16,::1/128,fc00::/7,fe80::/10,::ffff:7f00:1/104,::ffff:a00:0/104,::ffff:a9fe:0/112,::ffff:ac10:0/108,::ffff:c0a8:0/112
    - DOT_CACHING=on
    - BLOCK_MALICIOUS=on
```

---

# Starr Containers

I've included the paths for the backup directory in the *arr containers. If you don't want custom backup paths, just comment the line out. Their useful for centralized backup solutions, such as *arr containers -> backup directory -> rclone -> cloud directory / remote drive.

### [Prowlarr](https://hotio.dev/containers/prowlarr/)

Torrent monitor using RRS feeds. Only change `path/to/appdata` to your appdata folder. I kept the ports just in case, but when using GluetunVPN they are not necessary.

```yaml
 prowlarr:
    image: ghcr.io/hotio/prowlarr:latest
    container_name: prowlarr
    network_mode: container:GluetunVPN
    depends_on:
      GluetunVPN:
         condition: service_healthy
    environment:
      - PUID=99
      - PGID=100
      - UMASK=000
      - TZ=Europe/Stockholm
    volumes:
      - /path/to/appdata/prowlarr:/config
      - /path/to/backup/docker/prowlarr:/config/Backups
#    ports:
#      - 9696:9696
    restart: unless-stopped
```

### [Sonarr](https://hotio.dev/containers/sonarr/)

The TV series monitor and download manager that needs no introduction. Only things to change is `path/to/appdata` to point to your appdata folder and `/path/to/data` to where you store your media.

```yaml
 sonarr:
    image: ghcr.io/hotio/sonarr:latest
    container_name: sonarr
    network_mode: container:GluetunVPN
    depends_on:
      GluetunVPN:
         condition: service_healthy
    environment:
      - PUID=99
      - PGID=100
      - UMASK=022
      - TZ=Europe/Stockholm
    volumes:
      - /path/to/appdata/sonarr:/config
      - /path/to/data/:/data #optional
      - /path/to/backup/docker/sonarr:/config/Backups
#    ports:
#      - 8989:8989
    restart: unless-stopped
```

### [Radarr](https://hotio.dev/containers/radarr/)

Like Sonarr but for movies. Same things to change as Sonarr.

```yaml
 radarr:
    image: ghcr.io/hotio/radarr:latest
    container_name: radarr
    network_mode: container:GluetunVPN
    depends_on:
      GluetunVPN:
         condition: service_healthy
    environment:
      - PUID=99
      - PGID=100
      - UMASK=022
      - TZ=Europe/Stockholm
    volumes:
      - /path/to/appdata/radarr:/config
      - /path/to/data/:/data #optional
      - /path/to/backup/docker/radarr:/config/Backups
#    ports:
#      - 7878:7878
    restart: unless-stopped
```

### [Lidarr](https://hotio.dev/containers/lidarr/)

Searches for music to download and manages local music files.

```yaml
 lidarr:
    container_name: lidarr
    image: ghcr.io/hotio/lidarr
    network_mode: container:GluetunVPN
    depends_on:
      GluetunVPN:
         condition: service_healthy
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Etc/UTC
    volumes:
      - /path/to/appdata:/config
      - /path/to/data:/data
      - /path/to/backup/docker/lidarr:/config/Backups
#    ports:
#      - "8686:8686"
    restart: unless-stopped
```

### [Readarr](https://hotio.dev/containers/readarr/)

Manages and searches for books and comics to download.

```yaml
 readarr:
    container_name: readarr
    image: ghcr.io/hotio/readarr
    network_mode: container:GluetunVPN
    depends_on:
      GluetunVPN:
         condition: service_healthy
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Etc/UTC
    volumes:
      - /path/to/appdata:/config
      - /path/to/data:/data
      - /path/to/backup/docker/readarr:/config/Backups
#    ports:
#      - "8787:8787"
    restart: unless-stopped
```

### [Bazarr](https://hotio.dev/containers/bazarr/)

Monitors your media library and downloads subtitles for your movies and tv shows. It really hates being behind a VPN, that's why I chose bridged for its networking.

```yaml
 bazarr:
    container_name: bazarr
    image: ghcr.io/hotio/bazarr
    network_mode: bridged
    depends_on:
      GluetunVPN:
         condition: service_healthy
    environment:
      - PUID=1000
      - PGID=1000
      - UMASK=002
      - TZ=Etc/UTC
    volumes:
      - /path/to/appdata/bazarr:/config
      - /path/to/data:/data
      - /path/to/backup/docker/bazarr:/config/backup
    ports:
      - "6767:6767"
    restart: unless-stopped
```

---

# Downloaders

### [QBittorrent](https://hotio.dev/containers/qbittorrent/)

One of the many great downloaders. Just change the volumes to your liking.

```yaml
 qbittorrent:
    image: ghcr.io/hotio/qbittorrent:latest
    container_name: qbittorrent
    network_mode: container:GluetunVPN
    depends_on:
      GluetunVPN:
         condition: service_healthy
    environment:
      - PUID=99
      - PGID=100
      - UMASK=002
      - TZ=Europe/Stockholm
      - WEBUI_PORTS=8180/tcp,8180/udp
    volumes:
      - path/to/appdata/qbittorrent:/config
      - path/to/data/torrents/:/data/torrents #optional
#    ports:
#      - 8180:8180
    restart: unless-stopped
```

### [Deluge](https://hub.docker.com/r/linuxserver/deluge)

The classic Linux torrent downloader that's reliable and easy to use.

```yaml
  deluge:
    image: lscr.io/linuxserver/deluge:latest
    container_name: deluge
    network_mode: container:GluetunVPN
    depends_on:
      GluetunVPN:
         condition: service_healthy
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - DELUGE_LOGLEVEL=error #optional
    volumes:
      - /path/to/appdata/deluge:/config
      - /path/to/data/torrents:/downloads
#    ports:
#      - 8112:8112
#      - 6881:6881
#      - 6881:6881/udp
    restart: unless-stopped
```

### [Transmission](https://hub.docker.com/r/linuxserver/transmission)

A minimalist and easy to drive Torrent downloader.

```yaml
  transmission:
    image: lscr.io/linuxserver/transmission:latest
    container_name: transmission
    network_mode: container:GluetunVPN
    depends_on:
      GluetunVPN:
         condition: service_healthy
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - TRANSMISSION_WEB_HOME= #optional
      - USER= #optional
      - PASS= #optional
      - WHITELIST= #optional
      - PEERPORT= #optional
      - HOST_WHITELIST= #optional
    volumes:
      - /path/to/appdata/transmission:/config
      - /path/to/data/torrents:/downloads
      - /path/to/watch/folder:/watch
#    ports:
#      - 9091:9091
#      - 51413:51413
#      - 51413:51413/udp
    restart: unless-stopped
```

---
### [Doplarr](https://github.com/kiranshila/Doplarr)
If you've reached this far, I'm going to give you a little bonus. Every heard of Doplarr? Its a awesome little Discord bot that connects with Sonarr and Radarr and allow you to request media through a Discord server. Here's the code:

```yaml
doplarr:
  environment:
    - SONARR__URL=http://localhost:8989
    - RADARR__URL=http://localhost:7878
    - SONARR__API=sonarr_api
    - RADARR__API=radarr_api
    - DISCORD__TOKEN=bot_token
  container_name: doplarr
  image: "ghcr.io/kiranshila/doplarr:latest"
```

---

### Refrences

- GluetunVPN by qdm12 on Github  
	[https://github.com/qdm12/gluetun](https://github.com/qdm12/gluetun)
- List of containers by Linuxsserver.io  
	[https://fleet.linuxserver.io](https://fleet.linuxserver.io)
- List of containers by Hotio.dev  
	[https://hotio.dev/containers/autoscan/](https://hotio.dev/containers/autoscan/)
- Doplarr by karanshila on Github page  
	[https://kiranshila.github.io/Doplarr/#/](https://kiranshila.github.io/Doplarr/#/)