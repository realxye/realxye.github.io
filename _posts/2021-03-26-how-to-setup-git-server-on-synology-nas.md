---
layout: post
title: How To Setup Git Server On Synology Nas
author: Xiang Ye
description:
permalink: /:title/
categories: [Howto]
tags: [git, synology, nas]
---

This post explains how to setup git server on Synology nas.

- NAS: Synology DS418

## Steps ##

### Enable SSL ###

- Open `Control Panel` -> `Terminal & SNMP`
- Check `Enable SSH service`

### Install Git Server ###

The first step is to install Git Server Package.

- Open `Package Center` and search `Git Server`
- Install it

### Create Git Folder ###

This step is optional. You can use your main user to do this work.

- Connect to NAS via SSH
- Create git root folder: `sudo mkdir /volume1/git`

### Create User (optional) ###

This step is optional. You can use your main user to do this work.

- Open `Control Panel` -> `User`
- Create

### Config Git Server ###

#### Ensure Git Server Show Existing Users ####

In my case, git server complains that I haven't created any user although there are several users. This can be fixed by changing ile `"SYNO.Git.lib"`.

- Stop `Git Server`
- Connect to NAS via SSH
- Edit `"SYNO.Git.lib"` by exec command `sudo vi /var/packages/Git/target/webapi/SYNO.Git.lib`.
- Look for **"appPriv": "SYNO.SDS.GIT.Instance"**
- Remove the text between the quotes: **"appPriv": ""**
- Save and quit
- Run `Git Server` again
- Oen `Git Server` to ensure all users appear in git Server's Web GUI

#### Setup New Repo ####

Use following commands to create a new repo.

```bash
# Connect
ssh <user>@<nashost>
# Init new git repo
cd /volume1/git
sudo git init --bare --shared <repo-name>.git
# Set owner
sudo chown -R <user>:users <repo-name>.git
# Ensure owner have Write permission
chmod u+w <repo-name>.git
# Update
sudo git update-server-info
```

#### Clone Repo ####

- Clone `git clone ssh://<user>@<nashost>/volume1/git/<repo-name>.git`
- Try basic operations (`pull`, `push`, etc.) to ensure everything works.
