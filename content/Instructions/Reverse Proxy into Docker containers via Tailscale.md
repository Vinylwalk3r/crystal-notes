---
title: Reverse Proxy into Unraid Dockers via Tailscale
date: 2024-06-24
aliases:
  - reverse proxy tailscale
  - unraid dockers reachable over VPN
draft: false
tags:
  - VPN
  - Tailscale
  - reverse
  - proxy
  - docker
  - remote
  - access
  - peer to peer
  - cloudflare
  - cname
  - a record
---
Having a reverse proxy in a Tailscale network can be very useful of you have a lot of services and have a hard time remembering the ports for all your services. It also comes with the added bonus of removing the pesky "This site is not secure" warning since we'll be generating signed certificates for our domain.

>[!info] It's important to follow this guide step by step.
---
### Preparation
1. A custom Docker network to put Tailscale and all of the services we want to be able to access through Tailscale on. Below is my `Docker network create`  command:
```
sudo docker network create --driver=bridge --ip-range=100.10.0.0/24 --gateway=100.10.0.1 --subnet=100.10.0.0/24 tail-net
```
2. A Tailscale docker container on the custom Docker network we created above. I recommend either "EDACerton"s container for its continued (at the time of writing at least!) updates, but I've not had any problems running "deasmi"s container either. So whichever you prefer there.
3. A [Nginx Proxy Manager](https://hub.docker.com/r/jc21/nginx-proxy-manager) (hereafter referred to as NPM) Docker container. [NPM Plus](https://github.com/ZoeyVid/NPMplus) will also work well here. 
4. A domain that you'll use to access your services. Since I use Cloudflare, thats what I'll be using for this guide.
5. A Tailscale account
---
### Tailscale Container Config
> [!attention] Important
> In the TS container settings, add `--accept-dns=false` in the "Extra Arguments" field. [^1] 
1. If you have not done so already, set TS to use the custom docker network (in this example, Tail-net) you've created by changing its "Network Type"
2. Give the container whatever hostname you like using the provided field (this is the name the device will have in your "My Devices" list in Tailscale)
3. Follow the directions in the container to add the node to your account 
4. I recommend disabling key expiry for this node in your TS admin panel
---
### NPM Container Config
1. In the container config, toggle on "Advanced View" in the top right 
2. Change the "Network Type" to "None" 
3. In the "Extra Parameters" field add `--net=container:[name-of-TS-container]` 
4. Ensure the TS container starts before NPM by placing it higher in your list of Docker containers than NPM. There should be a little green lock icon on the right of your Unraid navigation bar that will let you rearrange containers after you click it. 
>[!warning] Tailscale FIRST! NPM Second!
>If NPM ever starts while TS is not running, it will go into a crash loop and you might have to disable autostart on the container and restart the Docker service to recover.
---
### NPM Config
1. Open your NPM web UI. You won't have any ports on your Unraid host to do this anymore, but that's not a problem, you can access it at the Tailscale address of your Docker node, port 81. The default login can be found in the overview in the container settings if you haven't already changed it.
2. Add a new admin user for yourself, log in using the new credentials, then delete the default one.
3. Go to the SSL certificates tab and click "Add SSL Certificate" (if you using NPM Plus, its called "TLS Certificate") to add a new Let's Encrypt (NPM Plus, certbot) cert.
4. I like using wildcard certs for this for simplicity, so I use "\*.example.com"; if you aren't sure about this, just use a wildcard cert.
5. Enter your email, toggle on "Use a DNS Challenge", toggle to agree to the ToS, then select Cloudflare as your DNS provider; the DNS challenge option is used because NPM is not running at a public IP address.
6. In the text box that shows up, paste the API token you copied down earlier in where the placeholder text is
7. Save it, and if it fails, try it again with longer propagation time; I've had to increase it to 30s in the past to get it to work for me.
---
### Tailnet Config
1. In the TS Admin panel, go into "DNS", then scroll down to "Nameserves". Under "Global Nameservers", add "Cloudflare Public DNS"
2. Change the "Override local DNS" toggle to ON
>[!info] Normally you don't want to use the "Override local DNS" function, but I couldn't get it to work any other way.
3. Now go to "Access Controls" and edit the `hosts` andd `acls` part to something like this:
```json
	// Declare conviniet hostnames to be used in place of IP adresses
	"hosts": {
		"[your device name]":       "[your devices Tailscale IP]",
	},
```

```json
"acls": [
		// Allow all connections.
		// Comment this section out if you want to define specific restrictions.
		{
			"action": "accept",
			"src":    ["autogroup:memeber"],
			"dst": [
				"[your device name]:*", // Allow access to all ports on [device]
				//Remove the comment below to enable access to exit nodes.
//				"autogroup:internet:*", // Allow access to Exit Nodes advertised on the network
			],
		},
	],
```
> [!attention] Tricky ACLs!
> A lot of Tailscale problems can stem from poorly configured ACLs! Always double check you ACL rules to make sure they work as intended. Errors can be really tricky to spot!
---
### Cloudflare Config
1. For the domain you want to use, set your A record  [^2] to point to your TS Docker node's address (get this by going into your TS Admin console and copying the devices "Adress" field) and disable Cloudflare's proxy; you don't need it. Anyone can look up the address, but it's a private IP that's only accessible to your Tailnet or those you've shared the node with.
2.  Create a zone edit token for your domain and copy it to a notepad. You create tokens in your Cloudflare profile, use the "Edit zone DNS" template and in the "Zone Resources" section, set it to **Include**, **Specific Zone**, **[Your Domain]**. The first two entries should already be set, so all you really need to do is set it to your domain.
---
### Extra Instructions for NPM
Each host obviously needs to be set up in Cloudflare as a CNAME (and remember, you don't want any of them proxied), but also in NPM. For NPM, you can use **the name of the Docker containers** as the destination address. 

The last thing to keep in mind is that when you set up your proxy hosts, you need to use the **internal port** the container is listening on, not whatever port you have mapped on the host because NPM is connecting directly to the containers, not through the host IP.

---
### Setting up CNAMEs in Cloudflare
The CNAMEs in Cloudflare can be tricky to set up for our Tailscale This is how I set up mine:

Type = `CNAME` 

Name (required) = `foo.bar`

Target (required) = `bar.example.com` or `bar.@`

Proxy Status = `OFF`

TTL = `Auto`

>[!info] Remember that we already have a A Record for "bar.example.com" so we only need to CNAME "foo.bar" to get it to work!
---
### Refrences
- HOW TO: Reverse Proxy with Tailscale on Reddit
	https://www.reddit.com/r/unRAID/comments/1darur5/how_to_reverse_proxy_with_tailscale/
- NPM Plus on Github
	https://github.com/ZoeyVid/NPMplus



[^1]: The "--accept-dns=false" flag in TS that was added earlier is to make sure that Docker host names keep working. Without that flag, TS may override the Docker DNS and those hostnames may not work depending on what settings you're using on your TS admin panel. Since the DNS is kind of irrelevant for this Docker node it's fine to disable it here. This was a detail that caused me a lot of headaches before I figured out what the problem was and how to solve it, so don't overlook it.

[^2]: You can have more than one A record on a domain, as long as they don't conflict with each other! Take my setup for example: "example.com -> A record (proxied) -> IP adress" and then "foo.example.com -> A record (DNS only) -> Tailscale IP". Then a bunch of CNAMEs pointing to those A records. This allows me to use only 1 domain but split it between Tailscale and the few services I expose to the web..
