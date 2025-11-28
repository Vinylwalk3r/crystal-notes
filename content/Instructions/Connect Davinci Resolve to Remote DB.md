---
title: Connect Davinci Resolve to Remote DB
date: 2023-11-04 12:32
aliases: 
draft: false
tags:
  - postgre
  - sql
  - remote
  - davinci
  - resolve
  - database
  - project
  - manager
  - extracting
  - docker
  - resolvedbkey
  - instruction
---
 
Before we get to into how to connect to a remote database in Resolve, please follow my guide on how to set up a dockerized PostgreSQL db for Davinci Resolve (or any other program that connects to PostgreSQL dbs)

[[PostgreSQL - Multiple DBs in One Instance - Compose]]

---

### Connecting to remote DB over LAN

Open Resolves __Project Manager__. Go to __Network__ -> __Add Project Library__. Select __Connect__.  
`Name` is the name of the database (This MUST be the name of the db in PostgreSQL. You can not set this to whatever you want independent of what is in PostgreSQL!).  
`Location` is the LAN IP of the computer hosting your DB. (You could __MAYBE__ get this to work over WAN using a VPN, but I haven't tried it and the performance would most probably be terrible anyway)  
`Username` and `Password` should be set to ****postgres**** if you followed my guide.

### Extract info from already connected DB

If you want to find out the connection info for a database that your already connected to, go to __Details__ for the database in question, then click __Share Key__ and save the ".resolvedbkey" file to easily accessible directory.  
Then open that file in a text editor.  
The IP address, database name, username and password used in connecting is all there.

---