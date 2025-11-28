---
title: Watch & Download Videos w. MPC HC
date: 2023-12-31 18:45
aliases: 
  - MPC HC Local Webplayer
draft: false
tags:
  - media
  - player
  - classic
  - home
  - cinema
  - mpc hc
  - yt-dlp
  - youtube
  - local
  - urls
  - online
  - videos
  - transcode
  - ffmpeg
  - player
  - instruction
---
 
I had no idea MPC HC could view videos via URLs until I read the changelogs for the latest version on their Github page (link below).So I wanted to try it myself.

### Installation

Download and install MPC HC from [their Github](https://github.com/clsid2/mpc-hc/releases).  
Then, download [yt-dlp.exe](https://github.com/yt-dlp/yt-dlp/releases) and [ffmpeg](https://www.gyan.dev/ffmpeg/builds/).  
After installation, on Windows, paste yt-dlp.exe and the ffmpeg folder into `C:\Program Files\MPC-HC`

### Use

You could just start using it now, as is. But you wont be able to view of download "Members Only" videos. If you want to be able to do that, follow along.

#### Members Only videos

Open MPC HC and go to "Options > Advanced". Scroll down to `YDL CommandLine` and paste this `-cookies-from-browser <your-browser-here>`. Type the name of your browser and press enter. (firefox, chrome, opera, vivaldi, etc).

Then press "Control+O" and input the URL of a video. DONT START IT YET! Close your browser so yt-dlp can grab you cookie file. Now you can start the video.

*Sidenote, this has been very problematic for me, so it may work for you, it may not. I'm still stoubleshooting!

### Extras

##### Extracting Cookies from Browser

Write this command to extract your browsers cookies to a cookies.txt file (which will be saved to your User folder):

`yt-dlp.exe --cookies-from-browser <your-browser-here> --cookies cookies.txt`

Then copy the "cookies.txt" file into the install path of MPC HC (usually `C:/Program Files/MPC-HC/` ).

##### Download with YT-DLP

To download a video without opening MPC-HC, open a cmd ****with administrator permissions**** (I got "Permission Denied" errors when using a non-admin cmd session) and paste this command:

```
yt-dlp ---cookies-from-browser >your-browser-here> <url>
```

Don't forget to close your browser before starting the download, otherwise you'll encounter errors.

### Refrences

- __Releases__ on MPC-HC Github  
	[https://github.com/clsid2/mpc-hc/releases](https://github.com/clsid2/mpc-hc/releases)
- __Releases__ on yt-dlp Github  
	[https://github.com/yt-dlp/yt-dlp/releases](https://github.com/yt-dlp/yt-dlp/releases)
- __ffmpeg builds__ on gyan.dev  
	[https://www.gyan.dev/ffmpeg/builds/](https://www.gyan.dev/ffmpeg/builds/)
- __YT-DLP "Cookies-from-browser"*__ Explained on Reddit 
	https://www.reddit.com/r/youtubedl/comments/14vv94s/comment/jremjh5/?utm_source=share&utm_medium=web2x&context=3](https://www.reddit.com/r/youtubedl/comments/14vv94s/comment/jremjh5/?utm_source=share&utm_medium=web2x&context=3)
- __Cookies__ on YTDL Wiki  
	[https://www.reddit.com/r/youtubedl/wiki/cookies/](https://www.reddit.com/r/youtubedl/wiki/cookies/)