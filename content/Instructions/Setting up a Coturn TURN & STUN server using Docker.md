---
title: Setting up a CoTURN server using Docker
date: 2026-10-18
aliases:
  - coturn server setup
  - stun turn server configs
draft: false
tags:
  - coturn
  - turn
  - server
  - docker
  - container
  - haven
  - mumble
  - tcp
  - udp
  - voice-chat
---
After having had a rough time to get my Coturn container working, I want to share what I had to do for it to work. This guide applies mostly to Coturn with Violet needing some small changes and I believe Eturnal needing even more drastic changes. 

## The Router / Firewall Settings
I run OPNsense so this guide will have the instructions specific to OPNsense but all the ports and protocols will be written for you to use in your own routers OS.

### Aliases
First, lets create a *Alias* for Coturns port range.

>[!info] Coturn Ports
>**TCP/UDP = 3478** 
>**UDP = 49100-49200**
>If you change the `min-port` and `max-port` fields to in the Compose file below, you MUST change the ports in the rules below!

Do do that, we go to *Firewall* -> *Alias* and click the *+* (Add)
Name it anything you like.
Then we change the `Type` to *Port(s)* and set our port range in the `Content` field as such:
*49100:49200*. 

### NAT Rules
Next, lets set up some Firewall rules to allow our TURN traffic to pass through the firewall. We're gonna need two rules.

To begin, go to *Firewall* -> *NAT* -> *Destination NAT* and add a new rule (*+*).
Then set these settings:

 **ALLOW Port 3478**
```
Interface = LAN, WAN
Protocol = TCP/UDP
Destination Adress = WAN adress
Destination Port = Single Port ; 3478
Redirect Target IP = Single Host ; {ip-of-your-server}
Redirect Target Port = 3478
Firewall Rule = Register Rule
```

**ALLOW Portrange 49100-49200**
```
Interface = LAN, WAN
Protocol = UDP
Destination Adress = WAN adress
Destination Port = {the-name-of-your-port-alias-create-before}
Redirect Target IP = Single Host ; {ip-of-your-server}
Redirect Target Port = {the-name-of-your-port-alias-create-before}
Firewall Rule = Register Rule
```

### Firewall Rules
Lets check if the NAT rules created associated Firewall rules. In my case, it didn't so I had to make them myself. Here they are:

**ALLOW Port 3478**
```
Interface = LAN, WAN
Interface (origin) = any
Action = Pass
Direction = Both
Protocol = TCP/UDP
Source = any
Source port = any
Destination Adress = Single Host ; {ip-of-your-server}
Destination Port = 3478
```

**ALLOW Portrange 49100-49200**
```
Interface = LAN, WAN
Interface (origin) = any
Action = Pass
Direction = Both
Protocol = UDP
Source = any
Source port = any
Destination Adress = Single Host ; {ip-of-your-server}
Destination Port = {the-name-of-your-port-alias-create-before}
```

And that should be it!

## The Docker Composes
Lets begin with Coturns own *compose.yaml*. We don't need to change it up to much.

You'll want to change the fields:
- `{your-external-ip}`
- `{your.sub.domain}`
- `{a-big-randomly-generate-secret}`

```yaml
  coturn:
    image: ghcr.io/coturn/coturn:latest
    container_name: coturn
    restart: unless-stopped
    network_mode: host
    volumes:
        - ./coturn-config/turnserver.conf:/etc/turnserver.conf
    command:
      - -n
      - --listening-port=3478
      - --no-tls
#      - --no-dtls
      - --external-ip={your-external-ip}
      - --realm={your.sub.domain}
      - --use-auth-secret
      - --static-auth-secret={a-big-randomly-generate-secret}
      - --min-port=49100
      - --max-port=49200
```

You can try to uncomment `--no-dtls` but I've had the best results with it commented out.

---

Next is a sample from my Haven instance, that connects to Coturn. This will have to customized or completely rewritten to how your container needs it. But the URL usually stays the same.

```yaml
services:
  your-service-using-coturn:
     environment:
       # TURN relay for voice over internet
      - TURN_URL=turn:{your.sub.domain}:3478?transport=udp
      - TURN_SECRET={a-big-randomly-generate-secret}       # Shared secret (coturn --use-auth-secret)
```

>[!warning] Declare the Protocol!
>See the `?transport=udp` at the end of the `TURN_URL` field? That took me a week to figure out! It was that little addition that finally got the TURN server to respond to Havens connection requests.

## Coturn Config

Next up is the *turnserver.conf* that coturn needs. Place this in a subfolder called "*coturn-config*".

```
listening-port=3478
# tls-listening-port=5349
external-ip={your-external-ip}
realm={your.sub.domain}

min-port=49100
max-port=49200

# TLS related
no-tls
no-dtls

use-auth-secret
static-auth-secret={a-big-randomly-generate-secret}

verbose
```

If you've gotten this far, the coturn config is mostly self explanatory. I've commented out the tls port since my setup doesn't need it, the coturn isn't exposed to the internet through it's own subdomain. Rather, it leeches off of a shared subdomain. Coturn is only accessed via another container (Haven) which sits behind Caddy, my reverse proxy. So I can't speak for setting a internet exposed Coturn instance but mine is reachable through Haven, me and my friends can connect to it and use voice chat flawlessly now.

## The Caddy File
The snippet below, from my Caddyfile, is edited mostly to get Haven to play nice but I'm including it here since the TLS parts might be important for other projects using a TURN / STUN setup with Caddy infront.

```yaml
{your.sub.domain} {
        reverse_proxy {container / ip}:{container-port} # Don't set this to Coturns own ip and port! Or, well, I haven't tried at least...   
        {
         header_up Host {hosts}
         transport http {
                tls
                tls_insecure_skip_verify
        }
    }
}
```


## Refrences
- Coturns Github
   https://github.com/coturn/coturn
- Violets Github (a alternative TURN server)
   https://github.com/paullouisageneau/violet
-  Eturnal.net (yet another alternative)
   https://eturnal.net