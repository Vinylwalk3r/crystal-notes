---
title: Preparing JWCS RL Streamer Setup
date: 2026-02-11 23:41
aliases:
  - jwcs obs setup
  - tournament streamer
draft: false
tags:
  - jwcs
  - instruction
  - rocket-league
  - discord
  - games
  - bot
  - streamer
  - social
  - broadcast
  - decks
  - obs
  - real-time
  - macros
---
## Foreword
This guide help you set up the JWCS Rocket League streamer setup. It will guide you through downloading and unpacking the zip package, preparing OBS and Streamer.Bot, setup up the Streamer.Bot Decks and setting up Rocket League.

# Initial Install
1. Download the zip containing all assets, OBS and Streamer.Bot from [Mediafire]()
2. Go to your *Downloads* folder and cut the archive.
3. Go to *This PC* > C:\ (Local Disk) and paste it.
4. Unzip the archive here. 
5. You'll get a new folder named "OBS_RL_Tournament_Setup". Go into it
6. From here, you can start OBS and Streamer.Bot using the "shortcuts" I've put there. Go ahead and launch OBS first.

# OBS
>[!Warning]- Some Files are Missing since you last used OBS!
>If you see this popup,  just click the *Search Directory...* button, then navigate to and select this
>folder: `Rocket League Tournament Assets` . Let it search and it will find everything (and then
>CRASH!...yes, it crashes for me **EVERY TIME** when I do this....idk why...) 
>
>![[obs-missing-files-popup.png]]

## -- Startup Check --
After you've started OBS and fixed any missing files, we shall go through and make sure that all the scenes look good. If all scenes pass inspection and doesn't report any errors, we continue.
 
 1. Go to the *Discord Audio* scene and make sure the audio is picked up. 
 
## -- Connecting Youtube --
1. Go to the top left, click on *Settings* button.
2. Then go to the *Stream* tab.
3. In the *Service* dropdown box, select **Youtube - RTMPS**
4. Click *Connect Account* and go through the login wizard
   >[!error] YT Login Info
   >Since the login details hasn't been reset yet it makes us unable to login to the stream account. Please contact @vinylwalk3r for more details.

# Streamer.Bot (SB):
## --  Connecting OBS --
1. Go to *Stream Apps* > *OBS Studio* then right click and select *Add*. 
2. Change these fields:
`Name` = The name of this OBS connection
`Password` = The password that is auto generated in OBS (we'll go fetch this later)
`Auto Connect on  Startup` = Yes
`Reconnect on Disconnect` = Yes
3. Click *Ok* and, under "Status" it should now say "Connected".

>[!tip]- Connectivity Status at a Glance!
>You can check the connection status of any connected service in the top right hand corner! Click the "Connected x/x" text to see a dropdown with the status off all the configured services.
## -- Youtube --

> [!Help]- Again, we don't have account access so contact @vinylwalk3r for help.

1. Go to `Platforms` > `Youtube`
2. Find the **Broadcaster Account**, then select *Sign in with Google*.
3. After having logged in, it should say "Connected" whilst showing the accounts name and icon.
## -- Decks --

(This is needed to get the control panels to work)
1. Press the *Log In* button at the bottom left corner. Create an account if you don't already have one.
>[!Warning]- AND SAVE THE LOGIN CREDENTIALS IN A PASSWORD MANAGER IF YOU DONT ALREADY!
>My recommendation is [Bitwarden](https://bitwarden.com)
2. In the top right hand corner, click on your *account icon* and, in the drop down menu, select *Decks*.

Now, for the moment, SB sadly doesn't let us export Decks. So I will have to teach you how to make your own. Please proceed to the next sub category of steps:

#### Making your own Decks
1. Click on *New Deck* in the top right hand corner:
`Name` = Whatever you want (JWCS Control Deck maybe?)
`Remote Connection` = Yes
`Public Access` = No (unless you want others, outside of your home, to get access to it. You MUST, ALWAYS send a link to the person for them to have access to the Deck!)
2. Click on any empty button space
3. Select which button type you want:
 - **Button** - Just a button, click it and it does stuff
 - **Toggle Switch** - On / Off button, I used these A LOT
 - **Status Indicator** - Currently bugged to the moon, skip these.
 - **Slider** - Cool but lacks functionality at the time of writing.
#### Toggle Switches
 1. Select *Toggle Switch*
 2. Set the *Title* to something indicating it's deactivated, for example "GAMEPLAY HIDDEN"
 3. Change the *Background Image* to one of the Monochrome images in the folder `/Rocket League Tournament Assets/SB Buttons/Unactive Scenes`
 >[!info] SB Buttons subdirectories contains Background Images for both Active and Inactive Scenes!
4. Set an *Icon* if you want to.
 5. Change the *Visual State* toggle to "On" and redo steps 1-4 but for the Active state ("GAMEPLAY SHOWN", for example).
 6. In the *Actions* tab, go down to the *Action* selection box and choose an action to trigger when we toggle this button **On**. Lets choose "OBS - RL Scene"
 >[!tip] The Dropdown list is searchable! Just start typing away!
7. Since we only want this to trigger when we **Activate** it, we don't need to choose any action for the "Toggle Off" action.
8. As a final thing, go into the *Settings* tab and give this button a good name in the *Name* text field. I like to give simple descriptive names, e.i "Gameplay Button".

There we go, repeat those steps for all the scenes you wanna be able to switch between.
And then, lets configure SB to switch the backgrounds to highlight the active one. 
#### Current Active Scene Highlighter
1. In Streamer.Bot, go to *Actions & Queues* > *Actions* 
2. Find the action named *OBS - Active Scene Finder*
3. Open up any action in the top row of actions and select change the *Deck* to your Deck, the *Item* to the button we just created and *State* to "0".
   (this is why we gave the button a good name earlier, it's easier to find in this list later)
4. Then go down and do the same to the actions in the *Switch Cases*, but set their *States* to "1". This will activate only the button that represent the active scene (ie Gameplay or Commentator).
#### Winner Buttons
To make the Match Winner and Final Winner buttons, we must use a normal Button.
1. Click on a empty button space
2. Select *Button*
3. Do all the settings from the [[#Toggle Switches]], except for the differing states. This button only has one state.
4. Go to the *Actions* tab. This is where we will make it ask for input.
5. Select you *Action*.
6. Under *Arguments* click *Add Argument* and add a "string". Set the *Name* to, either, `winner-name` if it's the Match Winner button, OR `final-winner` if it's the Final Winner button but leave the *string* empty.
7. Go down to *Inputs* and create and click *Add Input Argument*. Then choose *text*, match the *Name* to the name given to the *Argument* above (!) then write a descriptive question in the *Label* field. For example, "Who Won?" (thats the question I used).

Done! Now, if you click this button, it will bring up a text box and ask for your input, then pipe that into OBS and change the Winner Title!

#### Clearing Highlight Clips Between Matches
To help me keep the Highlight reels fresh, I created a SB *Action* called "Delete Old Highlights" which runs a tiny .bat file I wrote that help me. To use it, create a button that points to the SB *Action*. Easy as that. 

>[!Example]- Empty Highlights Dir.bat
>```
># This moves into the Highlights directory
>cd Highlights
># This calls "delete" to empty the current dir and does it without questions
>del *.* /Q
>```
>

# Rocket League

## -- Installing CustomUI --

## -- Using CustomUI --