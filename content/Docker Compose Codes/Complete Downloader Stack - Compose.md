---
title: Complete Downloader Stack - Compose
date: 2023-11-01 23:05
aliases:
  - arr stack
  - media downloader dockers
draft: false
tags:
  - docker
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
  - compose-codes
---
 The Docker compose codes below are a build-your-own-stack collection of already configured codes ready to be assembled to a stack or (with slight modifications) used independently.

A supplementary guide for how to configure these applications are available below:

[[Configure a Complete Downloader Stack]]

I you want to use any of these codes without the VPN, just remove this and it will work:

```yaml
depends_on:
  GluetunVPN:
     condition: service_healthy
```

---

### [GluetunVPN](https://github.com/qdm12/gluetun)

The backbone of the stack. All the traffic from the other applications will tunnel through this one. The settings you want to edit are the ones in "<value>" (OpenVPN or Wireguard).  
If you want to add other applications through the VPN, add their ports to the `ports` and `FIREWALL_INPUT_PORTS` variables. Then change the "network mode" in the new containers to `container:GluetunVPN` .

```yaml
  # Gluetun
  GluetunVPN:
    image: qmcgaw/gluetun
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
#      - 8388:8388/tcp # Shadowsocks
#      - 8388:8388/udp # Shadowsocks
#      - 51820:51820 # Wireguard Server
      - 9696:9696 # Prowlarr
      - 9705:9705 # Huntarr
      - 8989:8989 # Sonarr
      - 7878:7878 # Radarr
      - 8686:8686 # Lidarr
      - 8787:8787 # Readarr
#      - 5656:5656 # Kapowarr
      - 6767:6767 # Bazarr
      - 8180:8180 # QBittorrent
#       - 8112:8112 # Deluge
      - 11011:11011 # Cleanuparr
    volumes:
      - ./gluetunvpn_config:/gluetun
    environment:
      # See https://github.com/qdm12/gluetun/wiki
      - VPN_SERVICE_PROVIDER=<provider-name>
      - VPN_TYPE=<wireguard/openvpn>
      # Wireguard:
      - WIREGUARD_PRIVATE_KEY=<auth-key-from-vpn-provider>
      - WIREGUARD_ADDRESSES=<ip-from-your-VPN-provider>
      - SERVER_COUNTRIES=Sweden
      - WIREGUARD_MTU=1500
      # Updater
      - UPDATER_PERIOD=24h
      - UPDATER_VPN_SERVICE_PROVIDERS=<provider-name>
      # Firewall
      - FIREWALL_INPUT_PORTS=9696,8989,7878,8686,8787,6767,8180 # Prowlarr, Sonarr, Radarr, Lidarr, Bookshelf, Bazarr, QBittorrent
      # DNS
      - DNS_SERVER=on
      - DNS_UPSTREAM_RESOLVERS=cloudflare
      - DNS_UPSTREAM_RESOLVER_TYPE=dot
      - DNS_CACHING=on
      - BLOCK_MALICIOUS=on
      # Other
      - TZ=Europe/Stockholm
```

---

# Starr Containers

These compose codes are easy enough to set up. Just change the `/path/to/data` to where you store your media and, if you want to use it, the `/path/to/backup` to somewhere where you can easily back them up.
 If you don't want custom backup paths, just comment the line out. They are  useful for centralized backup solutions, such as:

`*arr containers -> backup directory -> rclone -> cloud directory / remote drive`


>[!tip]- Custom Themes in *arr Dashboards!
>If you want to use custom themes in the *arr apps, paste this Docker Mods code in the Environment variables section:
>```
>- DOCKER_MODS=ghcr.io/themepark-dev/theme.park:<name-of-*arr-app>
>- TP_THEME=hotpink
>- TP_COMMUNITY_THEME=false
>  ```

### [Prowlarr](https://hotio.dev/containers/prowlarr/)

Torrent monitor using RSS feeds. Only change `path/to/appdata` to your appdata folder. I kept the ports just in case, but when using GluetunVPN they are not necessary.

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

### [Huntarr](https://github.com/plexguide/Huntarr.io)
A nifty little search helper for Sonarr, Radarr and Readarr. Huntarr will randomly (or sequentially) order a search for x number of missing episodes or movies at set intervals. 
Just remember to go into the webui and configure it's connections to the *arr apps.

```yaml
  huntarr:
    container_name: Huntarr
    network_mode: container:GluetunVPN
    environment:
      - TZ=Europe/Stockholm
    volumes:
      - /path/to/appdata/Huntarr:/config:rw
    image: huntarr/huntarr:latest
```

### [Sonarr](https://hotio.dev/containers/sonarr/)

The TV series monitor and download manager that needs no introduction.

```yaml
  # Sonarr
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    #    image: ghcr.io/hotio/sonarr:latest
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
      - ./sonarr_config:/config
      - /path/to/data/:/data #optional
      - /path/to/backup/sonarr:/config/Backups
    #    ports:
    #      - 8989:8989
    restart: unless-stopped
```

### [Radarr](https://hotio.dev/containers/radarr/)

Like Sonarr but for movies. Same things to change as Sonarr.

```yaml
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    #    image: ghcr.io/hotio/radarr:latest
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
      - ./radarr_config:/config
      - /path/to/data:/data #optional
      - /path/to/backup/radarr:/config/Backups
    #    ports:
    #      - 7878:7878
    restart: unless-stopped
```

### [Lidarr](https://hotio.dev/containers/lidarr/)

Searches for music to download and manages local music files.

```yaml
  # Lidarr
  lidarr:
    image: ghcr.io/hotio/lidarr:pr-plugins
    # image: lscr.io/linuxserver/lidarr:latest
    container_name: lidarr
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
      - ./lidarr_config:/config
      - /path/to/backup/lidarr:/config/Backups
      - /path/to/data:/data
    # ports:
    #    - 8686:8686
    restart: unless-stopped
```

### [Readarr / Bookshelf](https://hotio.dev/containers/readarr/)

Manages and searches for books and audiobooks to download.
>[!warning] Since Readarr is as good as dead, I've moved over to Bookshelf. It's a drop in replacement that works just like Readarr.

```yaml
  # Bookshelf
  bookshelf:
    image: ghcr.io/pennydreadful/bookshelf:hardcover
    container_name: bookshelf
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
      - ./bookshelf_config:/config
      - /path/to/backup/dir/bookshelf:/config/Backups
      - /path/to/data:/data
    #  ports:
    #    - 8787:8787
```

### [Bazarr](https://hotio.dev/containers/bazarr/)

Monitors your media library and downloads subtitles for your movies and tv shows. It really hates being behind a VPN, that's why I chose bridged for its networking.

```yaml
  # Bazarr
  bazarr:
    image: ghcr.io/hotio/bazarr:latest
    #    image: ghcr.io/hotio/bazarr:latest
    container_name: bazarr
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
      - ./bazarr_config:/config
      - /path/to/data/data:/data
      - /path/to/backup/bazarr:/config/Backups
    #    ports:
    #      - "6767:6767"
    restart: unless-stopped
```

### [Kapowarr](https://github.com/Casvt/Kapowarr)

Comics and Manga downloader 
>[!info] I've not had any success with this....but if you've tried to find manga, you know what a unstandardized dumpster fire finding stuff is). 

```yaml
  kapowarr:
    container_name: kapowarr
    image: mrcas/kapowarr:latest
    network_mode: container:GluetunVPN
    depends_on:
      GluetunVPN:
        condition: service_healthy
    volumes:
      - /path/to/appdata/kapowarr/db:/app/db
      - /path/to/downloads/dir/books:/app/temp_downloads
      - /path/to/data/media:/comics
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
# Extras

### [Cleanuparr](https://github.com/Cleanuparr/Cleanuparr)

A lifesaver when you've got Huntarr all set up and working. Cleanuparr blocks malicious files and acts as the janitor on your download queue. Stalled or way to slow downloads cluttering up your downloader? Cleanuparr get rid of them and adds them to the *arr apps blocklist. It also cleans finished downloads that have seeded for a give time or reached a certain ratio!

```
  # Cleanuperr
  cleanuparr:
    container_name: cleanuparr
    image: ghcr.io/cleanuparr/cleanuparr:latest
    restart: unless-stopped
    network_mode: container:GluetunVPN
    #    ports:
    #      - 11011:11011
    volumes:
      - ./cleanuparr_config:/config
      # if you want persistent logs
      - ./cleanuparr_config/logs:/var/logs
      # if you want to ignore certain downloads from being processed
      - ./cleanuparr_config/ignored.txt:/ignored.txt
      # if you're using cross-seed and the hardlinks functionality
      - /path/to/torrents:/downloads
    environment:
      - TZ=Europe/Stockholm
```

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

### [Recyclarr](https://github.com/recyclarr/recyclarr)

If you've ever found yourself among [THRaSHs Guides](https://trash-guides.info) and thought "This seems nice but *HOW* am I going to import this into my starr apps?", then your in luck. After some config configurating, your starr apps will be kept up to date with THRaSHs settings without you ever having to think about it again. Set and forget.

```yaml
  recyclarr:
    container_name: recyclarr
    networks:
      - main-net
    environment:
      - TZ=Europe/Stockholm
      - CRON_SCHEDULE=@daily
      - RECYCLARR_CREATE_CONFIG=false
      - volumes:
      - /path/to/appdata/recyclarr:/config:rw
    user: 99:100
    image: ghcr.io/recyclarr/recyclarr:latest
```

---

### Refrences

- GluetunVPN by qdm12 on Github  
	[https://github.com/qdm12/gluetun](https://github.com/qdm12/gluetun)
- List of containers by Linuxsserver.io  
	[https://fleet.linuxserver.io](https://fleet.linuxserver.io)
- List of containers by Hotio.dev  
	[https://hotio.dev/containers/autoscan/](https://hotio.dev/containers/autoscan/)
- Huntarr by Admin9705 on Github
	 [https://github.com/plexguide/Huntarr.io](https://github.com/plexguide/Huntarr.io)
- Kapowarr by Casvt on Github
	 [https://github.com/Casvt/Kapowarr](https://github.com/Casvt/Kapowarr)
- Cleanuparr by Flaminel on Github
     [https://github.com/Cleanuparr/Cleanuparr](https://github.com/Cleanuparr/Cleanuparr)
- Doplarr by karanshila on Github page
	[https://kiranshila.github.io/Doplarr/#/](https://kiranshila.github.io/Doplarr/#/)
- Recyclarr by rcdailey on Github
	 [https://github.com/recyclarr/recyclarr](https://github.com/recyclarr/recyclarr)