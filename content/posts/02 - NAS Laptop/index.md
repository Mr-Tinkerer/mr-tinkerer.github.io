---
date: 2026-05-18T00:00:00
title: Turning the Laptop into a NAS
slug: Laptop NAS
description: I will be turning an old laptop into a NAS that fits my basic needs.
tags:
  - OpenMediaVault
  - Laptop
  - Server
  - Fixing
  - Hardware
  - Bash Script
toc: true
---

## The NAS Plan
So recently, I have successfully [turned an old laptop into a Proxmox server](/posts/old-laptop-into-proxmox-server).  However, I have only got the OS installed on the laptop. No VMs, and no LXCs containers on there. So this blog will be me changing that by stetting up a NAS (Network Attached Storage).

Now there are many options for me to use for the NAS software. I could go with a NAS OS that has all of the tools setup like [TrueNAS](https://www.truenas.com/). Or I could install a Linux Distro and manually install & setup the tools. I have decided to go with [OpenMediaVault](https://www.openmediavault.org/) (OMV). The reason why is that I didn't want to manually create the NAS from a linux distro (yet). However, going with something like TrueNAS wouldn't be possible, as their minimum requirements would take up most of the resources of the laptop.  

![](Pasted%20image%2020260512140559.png)
*The minimum requirements for TrueNAS Scale.<sup>[1]</sup>*

## Installing OpenMediaVault
So I have installed the ISO and uploaded it to Proxmox. I then created a New VM with 2 CPU cores, 4GB of RAM, the ethernet NIC that's connected to the switch, a disk with 10GB of storage, and another disk with 500GB of storage. You might be wondering why I put two disk on the VM. The reason why is that most NAS software requires a separate disk that's dedicated for the NAS OS. That disk cant be used to host the NAS shares. The other 500GB disk will be used for storing all of the NAS shares.

![](OMV%20Disk%20Layout.svg)
*A diagram of what the VM's storage looks like.*

After that, I booted up the VM, and got into the OMV installer. The first screen was to select what language to set the system to. Now it would be stupid to set the language to something you don't know anything about. Especially if that language's character set is completely different then a language you know.

![](Pasted%20image%2020260512232114.png)
*OpenMediaVault's installer in Russian.*

**I accidentally set the installer to Russian**. Now the smart thing here would be to reboot the VM to start all over, especially since the VM was only up for a minute. However for some reason, I've decided to continue on installing OMV in Russian.  So I've spent the new few minute going through the menu's without knowing what they are talking about. However, that doesn't mean that they weren't any messages that I had no clue about. Some of the words weren't fully translated to Russian, allowing me to have a clue of what's going on. 

![](Pasted%20image%2020260512232538.png)
*OpenMediaVault attempting to get a DHCP in Russian. Notice how DHCP isn't translated.*

For instance, I was able to tell that OMV tried to setup DHCP because DHCP isn't translated. Not only that, but in the following screen, I was able to deduce that it was trying to ask me what to do with the failed DHCP attempt because hostname wasn't translated.  


![](Pasted%20image%2020260512233011.png)
*OpenMediaVault asking the user what to do with the failed DHCP attempt.*

From that screen, I was able to successfully guessed the manual IP setup option. From there, I gave it a static IP, a subnet mask of `/24`, and no DNS and gateway. The reason why is that there is no router connected to the switch. Just the Proxmox Laptop and my personal device. So OMV wouldn't have any way of accessing the internet. Now if you read how [I've setup the Proxmox Laptop](/posts/), then you would have known that there is a way for me to connect the VM to the internet via the Wi-Fi dongle. However, I've decided against that, as I just want this NAS to only be accessible by me. 

After that, I was able to finagle my way through the Russian menus, and setup OMV.  I finally got to the part where OMV installer installs OMV. "Ah finally, the installer. Surely nothing will go wrong at this point," I thought to myself. 

![](2026-05-12-23h39m59s445.png)
*The Proxmox Laptop with a kernel panic.*

Yeah so installing OMV somehow caused Proxmox to have a kernel panic and crashed. After forcing the laptop to reboot itself, Proxmox came back alive. I've booted up the OMV VM to be greeted with the installer again. So I've reinstalled OMV with the language set to one that I actually know.
## The Gaslighting Browsers
After installing OMV, I was greeting to the login screen, as well as the greeter. The greeter has the OMV's IP address, and the default user login info. 

![](Pasted%20image%2020260512234644.png)
*OMV's greeter and login screen.*

However, I was confused on why the IP address in the greeter was different then what I assign to it. But I ignored it as I typed in `https://192.168.100.3` into LibreWolf (Firefox clone that focus on privacy<sup>[2]</sup>). But LibreWolf said that there's no website on that IP address. 

![](Pasted%20image%2020260512235228.png)
*LibreWolf failed to connect to OMV.*

"Ah, OMV must not have a self signed certificate to use HTTPS. Let me just removed the s from the URL," I thought to myself. Too bad that didn't work, as I got the same error message. I've tried HTTP and HTTPS with the `192.168.178.15` and the `192.168.122.1` IPs, even tho they're not on the same subnet as my personal device. And to no one surpised, those IPs addresses didn't work. I thought that LibreWolf might be breaking the website, so I tried again on Chromium (Open source engine that powers Google Chrome<sup>[3]</sup>). 

![](Pasted%20image%2020260513000024.png)
*Chromium was unable to access OpenMediaVault, regardless of HTTP or HTTPS.*

At this point, I thought that maybe I wasn't able to access the OMV's VM. However, that was quickly proven wrong as I was able to successfully ping the OMV's VM. I thought that maybe I messed up the install of OMV. So I reinstalled it with a different IP this time (`192.168.100.4`). That didn't fix anything. After a lot of headbanging, I was able to finally able to fix the problem. Turns out, both LibreWolf and Chromium were liars. You see, whenever I was trying to use HTTP, the browsers were automatically changing it to HTTPS, **without letting me know**. I had to enter in the HTTP URL multiple times for them to let me use HTTP.  But after that, I was able to access the login screen of OMV. After logging in with the default login of `admin` and `openmediavault`<sup>[4]</sup>, I was able to sucessfuly land in the homepage.

![](Pasted%20image%2020260513001317.png)
*The OpenMediaVault's login page.*

![](Pasted%20image%2020260514191123.png)
*OpenMediaVaults home page.*

## Setup the Drive
Now that OMV is installed, it's time to make the 2nd disk a NAS storage drive. To do that, I would need to put a file system on the drive. To do that, I can navigate to `Storage` -> `File Systems`.  From there, I can click the + button to create a new file system. From doing that, I can pick a file system from the drop-down list.
![](Pasted%20image%2020260514191815.png)
*The file systems' menu.*

![](Pasted%20image%2020260514192030.png)
*A list of supported file systems in OpenMediaVault.*

Now I am famillar with EXT4 and BTRFS, since I have used both of them in my experience using Linux as my daily driver. I have also used ZFS before as a NAS file system, but OMV doesn't support that<sup>[5]</sup>. I have heard of XFS before, but I don't fully understand it, and the pros/cons of it. The same can't be said with JFS and F2FS, as I have never heard of them until now. 

As for the file system I've went with, it was BTRFS. The reason why is that it has support for snapshots, self-healing, data compression, and more<sup>[6]</sup>. EXT4 doesn't have any of those features, which isn't a bad thing. While it sucks that EXT4 isn't as feature complete as BTRFS, it also doesn't have to worry about those features over complimenting things. So after a quick formatting the drive with BTRFS in single mode, it has been mounted into OMV while not being complicated.

![](Pasted%20image%2020260514193139.png)
*The 2nd drive formatted and mounted into OpenMediaVault.*

## Sharing is Caring
But in order for me to use that drive, I would need to make a shared folder. So I navigated to `Storage` -> `Shared Folders`, and click on the create new button. From there, I created a new shared called "NAS", put it in the root of the newly crated file system, and gave it the predefined permission set of `Admin: read/write, Users: read/write, others: no access`.

![](Pasted%20image%2020260515154027.png)
*The menu of shared folder in OpenMediaVault.*

![](Pasted%20image%2020260515154330.png)
*The shared folder creation screen with the settings applied to it.*

Now, I would need to attach that shared folder to a service for hosting folders. In OMV, there are two main ways for me to share a folder: NFS (Network File Sharing<sup>[7]</sup>) and SMB(Server Message Block<sup>[8]</sup>)/CIFS(Common Internet File System<sup>[9]</sup>). Both of these protocols will allow file sharing. The difference between them is how they operate under the hood. 

SMB was created by IBM in the 1980s for the DOS systems<sup>[10]</sup>. When Microsoft made MS-DOS, SMB became a core part of Windows OS<sup>[10]</sup>. Because of that, SMB is has been made to become the native to Windows<sup>[10]</sup>. Meanwhile, NFS was created by Sun Microsystems in 1984 for Unix-based systems<sup>[10]</sup>. This makes the file permission system that NFS uses identical to Unix-based systems<sup>[10]</sup>. There are other difference between the two, but the file permission system is the biggest one to understand, for now. Also, this doesn't mean that SMB can't be used on Unix-based systems, and NFS can't be used in Windows, as there has been tools made to allow that. 

With that in mind, I am going to share the folder with NFS, as I recently migrated my personal device to Linux ~~(I use arch, btw)~~. Not only that, but I've never used the NFS share before. I did have Linux machines that mounted a shared folder, but it was with SMB. The reason why was that I didn't understand NFS at the time, and I already had the SMB setup for my Windows machine, so I reused that. Anyway, enough yapping. In order for me to use NFS, I had to install the `nfs-utils` package, and enable the NFS client service<sup>[11]</sup>. Then I went to the `Services` -> `NFS` -> `Shares` section of OMV. From there, I created a new NFS share. I assigned it the NAS folder, set the allow network IP, and gave it the `Read/Write` permission.

![](Pasted%20image%2020260517204620.png)
*The NFS section in OpenMediaVault.*

![](Pasted%20image%2020260517225557.png)
*The NFS share setting.*

With the NFS share made, I just have to mount it. I created the `/mnt/OMV` directory to use for mounting the share. And with the `mount -t nfs 192.168.100.4:/ /mnt/OMV`, I was able to mount it.  Just remember to enable the NFS share beforehand, or else you'll waste 20 minutes trying to figure out why you can't mount it.

![](Pasted%20image%2020260517225715.png)
*NFS being disabled by default.*

![](Pasted%20image%2020260517225827.png)
*The NAS folder being accessible from my local machine.*
## It's Always Permissions Issues.
Now wait a minute, we're not done yet with the NAS. Even though I can access the NAS from my machine, I can't do anything with it, as I don't have permission to access it. 

![](Pasted%20image%2020260517233142.png)
*Unable to enter in the NAS shared folder, even as the root user.*

This is most likely because I haven't setup the correct permission for the share.  So the first thing I need to do is to create a new user for my device to use. So I've created an owner user with a "temporary" password for testing. I also add it to all of the groups that I think it would need to work.

Then I went to the permission of the NAS shared folder. From there, I gave the user read & write access. Did this allow me to access the NAS share? Nope, as I still get the same error.

![](Pasted%20image%2020260518003635.png)
*The permission button for the NAS share.*

![](Pasted%20image%2020260518003643.png)
*Giving the Owner user read/write access.*

![](Pasted%20image%2020260518003723.png)
*Still unable to access the NAS folder from my machine.*

I was confused on what to do next, as I was sure that it was a permission error. So I went back to the NAS share folder menu, and that's when I notice the ACL button. "That whats must be the issue," I thought to myself. And sure enough, when I click the button, I see that the file owner is set to root. 

![](Pasted%20image%2020260518004055.png)
*The access control list button for shared folders.*

![](Pasted%20image%2020260518004104.png)
*the NAS share folder's current access control list.*

After I switch it to the owner user, I was now able to access the NAS folder I was even able to create a test file in it.  I then unmount the NFS share. The reason why is that I wanted to remount it directory to the NAS folder, instead of mounting the entire `/export` directory. Doing it that way could cause permission issues in the future, due to how OMV setup the `/export` share options<sup>[12]</sup>.

![](Pasted%20image%2020260518004114.png)
*Obtaining permission to access the NAS folder, and creating a file in it.*

![](Pasted%20image%2020260518005014.png)
*Unmounting the share, and remounting with a direct path to the correct share.*

## Auto-mount that share!
Now from here, I could add in the share into the `/etc/fstab`, to make the it auto mount it, and call it a day. However, that wouldn't be optimal for me. The reason why is that my personal device is a laptop. So if I make it auto mount it from the kernel, then I would have to wait for it to mount it every time I boot up the laptop. Now Normally, this wouldn't be an issue, until I am waiting for my laptop to boot up because it's trying to connect to the NAS back home when I am in a **public coffee shop**.

So from my quick googling, I found a tool that will fix this issue for me. It is called autofs. What this tool does is that it will automatically mount the shares whenever it is being requested by an service, and auto unmount it when the share been's inactive<sup>[13]</sup>.  So I installed it to my laptop. Then, I went to set it up. Here's the thing, on Debian, Ubuntu, Fedora, and RHEL based distros, the config file for autofs is located at `/etc/auto.master`<sup>[14]</sup>. However, for Arch Linux,  it is instead located at `/etc/autofs/autofs.master`<sup>[15]</sup>. So just keep in mind that only Arch based distros have the autofs's files dumped into `/etc` instead of being a directory inside it. 

But regardless of distro you're using, the `autofs.master` behaves the same. The file will be modified to append the following line: `/mnt /etc/autofs/auto.nfs --timeout=60`. What this line does is that it tells autofs where to automatically mount the shares (`/mnt`), what shares to map to the parent directory (`/etc/autofs/auto.nfs`), and to automatically unmap the shares after 60 seconds inactivity (`--timeout=60`).  

Now the map file (`/etc/autofs/auto.nfs`) doesn't exist, so it needs to be manually created.  Then from here, you would add in the NFS shares you would want to be automatically mounted. `OMV -rw,soft,rsize=8192,wsize=8192 192.168.100.4:/export/NAS`. The first part would be the name of the directory that the share will be mounted **locally** (`OMV`). The second part is the NFS mounting options (`-rw,soft,rsize=8192,wsize=8192`). And the last part is the location of the NFS share (`OMV -rw,soft,rsize=8192,wsize=8192 192.168.100.4:/export/NAS`). 

From here, I would need to restart the autofs service for the new changes to be applied to it. Then I checked if the share is mounted. It isn't mounted, which is expected. But to see if autofs would mount it, I ran the `ls` command in the directory. And `ls` returned the file stored in the NAS, meaning that autofs automatically mounted it.  A quick verification proves that it is mounted.  And after waiting for a minute, the NFS share has been automatically unmounted.

![](Pasted%20image%2020260518013133.png)
*The NFS share being automatically mounted and unmounted.*

The problem that I didn't see is that I also had my Windows drive mounted in `/mnt/Windows`, causing it to be hidden by autofs. To fix this, I appended the following line to `auto.nfs`: `Windows -fstype=ntfs3,rw,uid=1000,gid=1000 :/dev/disk/by-uuid/A890A4F690A4CC5E`. But it's bothering me that there is a NTFS share in the `auto.nfs` file. Now I could have put that line in a seperate file (`auto.NTFS`), but then I wasn't sure if I could have one line in the `auto.master` file have more then one map file in it. So I renamed it to `auto.mnt`, updated that line, and append `--ghost` to that line so it would create the empty directories of them. After a quick restart of autofs, I was now able to see Windows in there.  
## Backup Automation
Now that the NAS is setup properly, I should actually use it. Right now, my only plans for this NAS is to backup my personal laptop. To be more specific, I am just going to backup my home directory, since that's where all of the files I care about live at. To do that, I will be using using a tool called Borg Backup. It is a free and open source tool that will copy the data given to it, compress the data, and encrypt the data<sup>[16]</sup>. Not only that, but any future backups will automatically remove any repeating data, saving tons of space<sup>[16]</sup>.  However, Borg does not do automatic backups, but I can take care of that later. One thing that's a downside about Borg is that it only supports UNIX devices, meaning that Windows is the only OS that it doesn't support<sup>[16]</sup>. 

The first thing I would need to do with Borg is to create a repo for the backups to live at. So I navigated to the NAS mount, and ran `borg init --encryption=repokey borg`. With the repo created, I can now start backing up data with it. So I used the following command to backup my home directory, excluding the cache, games, and the trashcan. I also added `notify-send` so that I would be notify when it's done, as it's going to take a while. And took a while it did, as I got the notification after 3 hours of waiting. 

``` bash
borg create \
    -e "$HOME/.cache" \
    -e "$HOME/.local/share/Steam/steamapps" \
    -e "$HOME/.steam" \
    -e "$HOME/Games" \
    -e "$HOME/.local/share/Trash/" \
    "/mnt/OMV/borg::$(date +%m-%d-%Y__%H-%M-%S)" \
    $HOME \
    --stats --progress && notify-send "The Borg backup is done."
```

![](Pasted%20image%2020260518175915.png)
*The borg backup stats.*

Now manually running the command would be lame, as I would have to remember to actually do that. That's why I'm going to automate it. The first thing I would need to do to automate it is to have a script that it will run. So I wrote a bash script (without using AI) that will backup the home directory, and notify me as well.

```bash
#!/bin/bash

#get the borg directory
REPO="/mnt/OMV/borg"

#get the name of the current archive
ARCHIVE_NAME=$(date +%m-%d-%Y__%H-%M-%S)

#the maxinium amount of backups to keep before it start to delete the older ones.
MAX_BACKUPS=10

#stop if the borg repo doesn't exist
if [[ ! -e $REPO/data ]]; then
    notify-send "Auto Borg Backup Error" "the Borg repo path is invalid. Make it first before running this script"
    exit 1
fi

#get the file that's storing the borg secret
SECRET_FILE="./BORG_SECRET"

#check if the file exists
if [[ -e $SECRET_FILE ]]; then
    #check if the file is readable, and has content inside it
    if [[ -r $SECRET_FILE ]] && [[ -s $SECRET_FILE ]]; then
        #get the password and set it to 400 for security
        export BORG_PASSPHRASE=$(cat $SECRET_FILE)
        chmod 400 $SECRET_FILE
    else
        #notify the file error
        notify-send "Auto Borg Backup Error" "The $SECRET_FILE needs to be readable, and have the password inside it."
        exit 1
    fi
else
    #notify the missing file error
    notify-send "Auto Borg Backup Error" "The $SECRET_FILE file needs to exist in order for the script to work."
    exit 1
fi

#the borg file that will be used to store the stats of the backup 
BORG_FILE=borg_results.txt

#start the borg backup while ignoring the cache, games, and the trash directory.
#also redirect the output to a file for parsing. 
notify-send "Starting Auto Borg Backup..."
(borg create \
    -e "$HOME/.cache" \
    -e "$HOME/.local/share/Steam/steamapps" \
    -e "$HOME/.steam" \
    -e "$HOME/Games" \
    -e "$HOME/.local/share/Trash/" \
    "$REPO::$ARCHIVE_NAME" \
    $HOME \
    --stats 2>&1) > $BORG_FILE

#get the stats from the borg backup file and delete it
DURATION=$(grep "Duration" $BORG_FILE | awk -F ' ' '{print $2, $3}')
CURRENT_STORAGE=$(grep "This archive" $BORG_FILE | awk -F ' ' '{print $7, $8}')
TOTAL_STORAGE=$(grep "All archives" $BORG_FILE | awk -F ' ' '{print $7, $8}')
rm $BORG_FILE

#notify that it's done with stats
notify-send "Auto Borg Backup Done in $DURATION." "This borg backup has taken up $CURRENT_STORAGE out of $TOTAL_STORAGE."


#check if there is more backups then the maxinium
BACKUPS=$(borg list $REPO | wc -l)
if [[ $BACKUPS -gt $MAX_BACKUPS ]]; then 
    #notify about the backups higher then the max
    notify-send "Auto Borg Backup Prune." "There is currently $BACKUPS out of $MAX_BACKUPS maxinium allowed. Going to delete the old one to make space."
    
    #delete the older backups to make the backups under the max
    borg prune --verbose --keep-last $MAX_BACKUPS $REPO

    #get the new size of the borg repo 
    SIZE=$(du -sh $REPO | awk -F ' ' '{print $1}')

    #notify that it's done, and say the new size.
    notify-send "Auto Borg Backup Prune Done." "Sucessfully prune the old backups. The repo is now $SIZE."
fi
```
*[The link to the Github Gist of this script.](https://gist.github.com/Mr-Tinkerer/e9748457eb8958dde768ad0ac3b25a54)*

Now if you want to use this script, note that you would have to change the borg's repo path, place the borg's secret inside the `BORG_SECRET` file, set that file to be that you're the only one that can read it (`chown 1000:1000 BORG_SECRET && chmod 400 BORG_SECRET`), and exclude/include any paths in the backup command. Now running that script should automatically create a new backup without any user input.

![](Pasted%20image%2020260518181500.png)
*The script being executed for 46 seconds.*

![](Pasted%20image%2020260518181514.png)
*The notifications that the script sent.*

Now this script is great and all, but I am still running it manually. You can't trust a monkey like me to manually run the script. That's why I am instead going to use systemd's timers to run the script for me. The reason why I am using systemd's timer instead of crontab is that crontab won't trigger the script when the time passed while the machine was off, while systemd tiemrs can do that. To setup systemd's timers, I would first need to make a new service called  `~/.config/systemd/user/auto_backup.service`. This is to make the systemd service run in user level so that the notification can work. Just note that I put the script in my `Documents/scripts `directory, so you would need to change that if you want to use this service. You would also have to check if your environments variables for `DISPLAY` and `DBUS_SESSION_BUS_ADDRESS` are the same, since they're needed for the notifications to work<sup>[17]</sup>. 

```toml
[Unit]
Description=Automatically backup the home directory via Borg.
After=graphical-session.target

[Service]
Type=oneshot
WorkingDirectory=/home/<YOUR_USERNAME>/Documents/scripts
ExecStart=/bin/bash borg_backup.sh
Environment="DISPLAY=:0" "DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus"

[Install]
WantedBy=graphical-session.target⏎
```

But the service isn't enough to make it automatic. I would need systemd's timer as well. So in the same directory as where I made the `auto_backup.service`, I made the `auto_backup.timer`. This timer will run the service (which has the script) every 6th hour of the day, even if the machine was off at that time. Now obviously, you can adjust the times to fit your needs. 

```toml
[Unit]
Description=Auto backup the home directory via Borg

[Timer] 
OnCalendar=*-*-* 06:00:00
OnCalendar=*-*-* 18:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

After that, systemd user's daemon will need to be reload, via `systemctl --user daemon-reload`, for it to see the new files. Finally, the timer would need to be enabled for it to be activated via `systemctl --user enable auto_backup.timer`. You can run `systemctl --user list-timers` to check when the next time the timer is going to be run, or you can run `systemctl --user start auto_backup.timer` to manually trigger the timer.

## Citations
> [1] TrueNAS. “SCALE Hardware Guide.” _Truenas.com_, TrueNAS, 3 Sept. 2024, www.truenas.com/docs/scale/24.10/gettingstarted/scalehardwareguide/. Accessed 12 May 2026.
> [2] LibreWolf. “LibreWolf Browser.” _Librewolf.net_, 7 Mar. 2020, librewolf.net/#what-is-librewolf. Accessed 13 May 2026.
> [3] Google Inc. “The Chromium Projects.” _Chromium_, Google Inc, 8 Sept. 2008, www.chromium.org/chromium-projects/. Accessed 13 May 2026.
> [4] OpenMediaVault. “FAQ — Openmediavault 7.X.y Documentation.” _Openmediavault.org_, 9 Nov. 2018, docs.openmediavault.org/en/stable/faq.html#what-are-the-default-credentials-for-the-ui. Accessed 14 May 2026.
> [5] OpenMediaVault. “Filesystems — Openmediavault 8.X.y Documentation.” _Openmediavault.org_, 9 Mar. 2025, docs.openmediavault.org/en/latest/administration/storage/filesystems.html#id1. Accessed 14 May 2026.
> [6] BTRFS. “Introduction — BTRFS Documentation.” _Readthedocs.io_, btrfs.readthedocs.io/en/latest/Introduction.html#introduction. Accessed 14 May 2026.
> [7] Wikipedia Contributors. “Network File System.” _Wikipedia_, Wikimedia Foundation, 22 Sept. 2019, en.wikipedia.org/wiki/Network_File_System. Accessed 17 May 2026.
> [8] Wikipedia Contributors. “Server Message Block.” _Wikipedia_, Wikimedia Foundation, 9 July 2019, en.wikipedia.org/wiki/Server_Message_Block.
> [9] Wikipedia Contributors. “CIFS.” _Server Message Block_, Wikimedia Foundation, 9 July 2019, en.wikipedia.org/wiki/Server_Message_Block.
> [10] AWS. “NFS vs SMB - Difference between File Access Storage Protocols - AWS.” _Amazon Web Services, Inc._, Amazon, aws.amazon.com/compare/the-difference-between-nfs-smb/. Accessed 17 May 2026.
> [11] Arch Wiki Contributors. “NFS - ArchWiki.” _Wiki.archlinux.org_, Arch Linux, wiki.archlinux.org/title/NFS. Accessed 17 May 2026.
> [12] OpenMediaVault Docs. “NFSv4 Pseudo filesystem” _NFS — Openmediavault 8.X.y Documentation_, OpenMediaVault, 2018, docs.openmediavault.org/en/latest/administration/services/nfs.html#nfsv4-pseudo-filesystem. Accessed 17 May 2026.
> [13] The Linux Foundation. “Autofs - How It Works — the Linux Kernel Documentation.” _Kernel.org_, 2024, www.kernel.org/doc/html/latest/filesystems/autofs.html. Accessed 18 May 2026.
> [14] Josphat, Mutai. “How to Automatically Mount NFS File System with AutoFS [Guide].” _Computing for Geeks_, 16 Mar. 2026, computingforgeeks.com/how-to-automatically-mount-nfs-file-system-with-autofs/. Accessed 18 May 2026.
> [15] Arch Wiki Contributors. “Autofs - ArchWiki.” _Wiki.archlinux.org_, Arch Linux, wiki.archlinux.org/title/autofs. Accessed 18 May 2026.
> [16] Borg Maintainers. “BorgBackup – Deduplicating Archiver with Compression and Authenticated Encryption.” _Borgbackup.org_, 2024, www.borgbackup.org/. Accessed 18 May 2026.
> [17] Cirelli94. “Systemd Service Not Executing Notify-Send.” _Stack Overflow_, 2 Apr. 2018, stackoverflow.com/a/66879066. Accessed 18 May 2026.
