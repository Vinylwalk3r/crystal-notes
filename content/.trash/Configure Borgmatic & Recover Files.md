---
title: Borgmatic Setup & Recovery
date: 2024-04-28
aliases: 
  - Restore from Borgmatic Backup
  - Configure Borgmatic Backups
draft: true
tags:
  - borgmatic
  - config
  - setup
  - restore
  - backup
  - recover
  - commands
  - repository
---
 
## Borgmatic Config

  

### Config Files

  

### Apprise Notifications

## Recover Files from Backup

This can be done several ways. One way is just to let Borgmatic overwrite the existing files on the host. Say you have a directory "test" that you have backed up. Then it gets corrupted. You CAN let Borgmatic just write into the "test" directory and overwrite the corrupt files with the good copies from the backup.  
OR  
We ask Borgmatic to pull the backed up files and put it in a different directory. I personally like this approach. This is how I do it:

```bash
borgmatic extract --repository {the name of the repo from "borgmatic rlist"} --archive [latest] --destination {destination path on host} --path {full path in archive}
```

That's a handful, so lets break it down.

- `extract` self explanatory. Borgmatics Extract operation.
- `--repository` specifies which repository you want to extract from.
- `--archive` specifies which archive (specific backup job) we want to get our files from. A commonly used tag is "latest", to pull the most up-to-date files.
- `--destination` tell Borgmatic where to put the extracted files. My Docker mapping is "/mnt/recover". Make sure it's a WRITABLE DIRECTORY for the Docker container.
- `--path` Optional. Use this to specify which directory / files you want to have extracted, if you don't want every file / folder. Must be written for every file / folder you want to have extracted as a complete path. For example:  
    "--path /mnt/user/adam/restore/this/file.1 --path /mnt/user/adam/restore/that/dir/"

### Refrences

- Borgmatic Documentation and Examples on Torsions.org  
	[https://torsion.org/borgmatic/](https://torsion.org/borgmatic/)