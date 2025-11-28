---
title: Create Semi-advanced Systemd Services
date: 2024-07-20
aliases:
  - systemd
  - create autostarted systemd services
  - pass environment variables to systemd
draft: false
tags:
  - systemd
  - services
  - system
  - environment
  - variabels
  - autostart
---
 
## Check if you run Systemd
You can easily check if you have systemd on you system using this command:
`systemctl`
If you get some output from that, you may proceed.

## Create the .service file

>[!note] Systemd service files resides in "/etc/systemd/system/"

Systemd identifies the services it has by checking which .service files exist in "etc/systemd/systems". So if we want to create a new service, we have to make a new ".service" file.

Create a new service file:
`sudo nano /etc/systemd/system/exampleservice.service`
>[!note] You may use nano, vim any other text editor of your choosing throughout this guide! 

A empty file will open. This is where we declare our service. It's name, path to its executable and some other parameters. We wont be passing environment variable to it yet though!

Below is a example .service file:
```
[Unit]
Description= FOO, the baariest of bars

[Service]
ExecStart=/path/to/your/executable
EnvironmentFile=/etc/my_service/my_service.conf
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

`Description` = This directive can be used to describe the name and basic functionality of the unit. It is returned by various `systemd` tools, so it is good to set this to something short, specific, and informative.

`ExecStart` = Path to your executable

`EnvironmentFile` = The path to an environment file

`Restart` = This indicates the circumstances under which `systemd` will attempt to automatically restart the service. This can be set to values like “always”, “on-success”, “on-failure”, “on-abnormal”, “on-abort”, or “on-watchdog”. These will trigger a restart according to the way that the service was stopped.

`WantedBy` = The `WantedBy=` directive is the most common way to specify how a unit should be enabled. This directive allows you to specify a dependency relationship in a similar way to the `Wants=` directive does in the `[Unit]` section. The difference is that this directive is included in the ancillary unit allowing the primary unit listed to remain relatively clean. When a unit with this directive is enabled, a directory will be created within `/etc/systemd/system` named after the specified unit with `.wants` appended to the end. Within this, a symbolic link to the current unit will be created, creating the dependency. For instance, if the current unit has `WantedBy=multi-user.target`, a directory called `multi-user.target.wants` will be created within `/etc/systemd/system` (if not already available) and a symbolic link to the current unit will be placed within. Disabling this unit removes the link and removes the dependency relationship.

>[!caution] Reload to find your service!
>Dont forget to reload the Systemctl Daemon after creating your new service! Do this using: `systemctl daemon-reload`
## Passing Environment Variables to service

##### Method A
>[!warning]
>Note that this method will allow anyone capable of running `systemctl show my_service` to see the content of the override.conf file! Using this method with caution.

This method is mostly used for passing non-sensitive environment variables to the service. This is done by running `systemctl edit myservice`, which will create an override file for you or let you edit an existing one.

In normal installations this will create a directory `/etc/systemd/system/myservice.service.d`, and inside that directory create a file whose name ends in `.conf` (typically, `override.conf`), and in this file you can add to or override any part of the unit shipped by the distribution."
 
To create a override file to edit, run: 
`systemctl edit exampleservice.service`

Below is a outtake from on one of my override.conf files:
```
[Service]
Environment="YOUR_VARIABLE=foo"
Environment="YOUR_SECOND_VAR=bar"
Environment="path/to/file"
```

#### Method B
A more secure way to pass environment variables is by using a independent env file that is only accessible by the service itself. This is done by:
1. Creating a config file using `nano /etc/example_service/exampleservice.conf`
2. Setting a `EnvironmentFile` in the [[Create Semi-advanced Systemd Services#Create the .service file|.service example]] file.

## Running your service
Now, start your service using:
`systemctl start exampleservice.service`
It should not return any output. 

Second, lets check it's health and status using:
`systemctl status exampleservice.service`
It should say "Active: active (running)"
Here you can also view logs from the service.

Thirdly, lets enable autostart of the service:
`systemctl enable exampleservice.service`
Authenticate and it shouldn't return output either. But we can check if it worked using `systemctl status exampleservice.service` and look at the second line that says "Loaded".  After the exec path, it should say "*enabled*", which tells us that it worked.


### Refrences
- "Understanding Systemd units and files" on DigitalOcean - https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files
- "Make exec run as service" on StackExchange - https://unix.stackexchange.com/questions/478999/how-can-i-make-an-executable-run-as-a-service
- "How to set environment variable in systemd service" on serverfault - https://serverfault.com/questions/413397/how-to-set-environment-variable-in-systemd-service