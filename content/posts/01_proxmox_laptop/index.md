---
date: 2026-05-20T14:34:00
title: Turning an Old Laptop into a Proxmox Server
slug: Old Laptop Into Proxmox Server
description: I try to turn an old laptop into a Proxmox server, but struggle in the process due to factors out of my control.
tags:
  - Proxmox
  - Laptop
  - Server
  - Fixing
  - Hardware
  - Lots of Issues
toc: true
---

## The Old Laptop
So I was recently talking to an classmate of mine when he mention that he has an old, semi-broken, laptop that he doesn't know what to do with it. Being someone who recently lost their home lab due to moving, I took that opportunity to rebuild my home lab. The laptop is an Asus x555q with an an AMD A12-9720P APU with 8GB of RAM, and 256GB SATA SSD<sup>[1]</sup>. 

![](Pasted%20image%2020260307124029.png)
*The laptop model.* <sup>[1]</sup>

However, he has upgraded the laptop to have 16GB of RAM, and 1TB of storage before he replaced it in 2022. When he gave me the laptop, I didn't notice anything at first that would make it semi-broken. However, he told me that the battery only last for about 40 minutes, and the left side of the keyboard doesn't register the key presses. Now I am fine with the battery not being good, since I am going to keep this plugged in. But the broken keyboard could be an issue worked around by using an external keyboard. 

When I gotten the laptop, I notice that he reinstalled windows 10 on it to "wipe it" for me. So I use that moment to look at the laptop specs in the performance tab of task manager. That's where I notice the CPU is an APU (Accelerated Processing Unit) chip, which is a chip that is both a CPU and a GPU. Now I never messed with an APU chip before, but I assume that it shouldn't be too much of an issue. After all, this laptop was released quite a while ago, so there has to be drivers and chipsets for this in the Linux Kernel by now.

## The Sneaky Issue
Since my plan is to install Proxmox OS onto the laptop, the first thing I do is install a Ubuntu onto it. The reason why is to ensure that everything important with this laptop works with Linux. I installed the Ubuntu 24.04 desktop version, and flash it into the USB stick. The reason why I went with the desktop version is that I wanted to use the live boot feature to test the hardware before installing it.

So I got into the desktop screen and attempted to connect this laptop to the network via Wi-Fi. Now I know that having a server being hosted over Wi-Fi is a bad idea for stability. But the thing is, I don't have any other choice. I am currently living in a dorm, so I don't have an Ethernet cable, nor I have access to the router. But it doesn't matter anyway cause I realize that Ubuntu's (or GNOME's) UI doesn't have support for connecting to an dorm Wi-Fi. The reason why is that the dorm Wi-Fi uses an WPA2 Enterprise, which means I have to type in my username and password from the password connect screen.  

![AN EXAMPLE OF CONNECTING TO A WPA2 ENTERPRISE WI-FI FROM KDE.](Pasted%20image%2020260311113331.png)
*An example of connecting to a WPA2 Enterprise Wi-Fi from KDE.*

I then re-flashed the USB stick with Kubuntu 24.04 instead. I know that the KDE has support for WPA2 Enterprise Wi-Fi, as my personal laptop uses it ~~(I use Arch, btw)~~ . And sure enough, the laptop is able to connect to the Wi-Fi. After that, I open up Firefox and navigate to [google.com](https://google.com).  However, Firefox isn't able to load the web-page, as it says that I don't have any connection. At first, I thought I misread the connection message, and I mistype in my login info. But when I went to reconnect the Wi-Fi, there was no Wi-Fi showing up. "Weird", I thought to myself.  I rebooted the system to see if that fixed the issue. And at first, it did, allowing me to successfully reconnect back to the Wi-Fi. But as soon as I attempt to load a web-page, the same issue appears.  This made me realize that there's a weird issue happening. I rebooted the system again, reconnect to the Wi-Fi, ping [google.com](https://google.com) from the terminal, and it stops pinging after a few packets. 

So something is definitely wrong with the Wi-Fi card. But maybe this version of Ubuntu had a broken driver with the Wi-Fi card. So I've flashed EndeavourOS, which is Arch Linux that's been setup beforehand. Since Arch Linux's package are always the latest version, the Wi-Fi drivers shouldn't be on the broken version, in theory. The reality is that it didn't changed anything. At this point, I started to think that the Wi-Fi chip is broken with Linux. To ensure that isn't the case, I reinstalled Windows 10 to see if that's the case. Since the laptop came with Windows 10, it should work just fine with it. Too bad that's a lie, since the same issues appeared on Windows 10. At this point, it has to be a hardware issue, since the Wi-Fi cared hated all of the OSes I tried it on. 

## Taking it apart
With the broken Wi-Fi card inside the laptop, I had no choice but to replace it with a new one. So I attempted to open up the laptop case to get into the components. Now this is my first time opening a laptop, so I was careful on opening it up.  However, since this is a semi broken laptop that was given to me, and not my daily driver laptop, the risks were low enough for me to do it. So with a iFixit tool kit, and a [disassembly guide](https://www.youtube.com/watch?v=kpkZ6dwhnB0) on hand, I started taking apart this laptop. 

After unscrewing the bottom screws, I got to the part where you take a "knife" (I don't know the official name) and stab the laptop with it to pry it open. Now this was the part I was most nervous about, since I didn't want to accidentally break the case. However, when I start prying the laptop open, I quickly realized that it wasn't so hard to get it open, especially since I notice that the "knife's" purpose was to disconnect the keyboard's clips. After I pry all sides of the keyboard, I was able to disconnect keyboard form the laptop, but I had to removed the laptop's ribbon cable to do so successfully. I'm just glad that I slowly moved the keyboard away from the laptop, or else I would had accidentally ripped the ribbon cable apart. 

![Pasted%20image%2020260315111542.png](Pasted%20image%2020260315111542.png)
*The keyboard's ribbon cable that was still connected to the laptop.<sup>[2]</sup>*

![Pasted%20image%2020260315111851.png%20|%20750](Pasted%20image%2020260315111851.png%20|%20750)
*The internals of the laptop.<sup>[2]</sup> (Text edited in by me)*

Now that I am inside the laptop, I have to find the Wi-Fi chip. After analyzing the video to find the Wi-Fi chip, I found something that might look like the Wi-Fi chip. But the problem is that the suspected chip is on the underside of the motherboard. And in order for me to get to it, I would have to take apart everything else.  
![Pasted%20image%2020260315112522.png](Pasted%20image%2020260315112522.png)
*The suspected Wi-Fi chip.<sup>[2]</sup>*

So the very first thing that I did was take out the battery, since I don't want it to explode on me while prying around the laptop. It was easy to take apart, since all I had to do was unscrew the screws around it and slide it out. Next was the CD drive, which was the same story as the battery. Unscrew the single screw and slide it out. 

Next was the hard drive, which required more steps, due to it being split into multiple parts. First, I had to disconnect the ribbon cable that connected the right side's ports to the motherboard. Then I had to unscrew the component that bridge the right side ports, the hard drive, and a random cable to the motherboard. After that, I had to unscrew the component that held the right side ports. Then, I take out the right side component first.  Finally, I can take out the hard drive and the bridge together. 

All that is left is the motherboard. I unscrew all of the screws that were connected to the motherboard, and unplug the display's internal cable. But as I attempt to take it out, I felt some resistance from the board. I immediately stop myself from taking it out, and I don't want to accidentally snap the motherboard into pieces. At first, I thought the USB ports on the motherboard was stuck in the case. So I tried to slide it out of there, but the motherboard wouldn't move. I was so confused as the [disassembly video](https://www.youtube.com/watch?v=kpkZ6dwhnB0) didn't had any issues taking it out. But then I realized that I missed a screw on the motherboard. After I unscrew that screw, I was able to take the motherboard out with no issues.

After flipping the motherboard around and taking apart the suspected Wi-Fi card, I was able to confirm that the suspected Wi-Fi card was in fact, the Wi-Fi card. After I was able to properly read the text on the card, I attempted searched up the Wi-Fi card. However, I couldn't find the listing online.  But the thing is, past me doesn't know how to use google, as I was able to easily find the Wi-Fi chip [online](https://www.amazon.ca/MPE-AXE3000H-AX210-AX3000Mbps-Bluetooth-Adapter/dp/B0DP9YKVRL)<sup>[3]</sup>. But, Either the listing is unavailable, or it wouldn't shipped in time. So being able to find the listing wouldn't changed anything I did in the past.

After being disappointed that I couldn't find the chip, I moved on to plan B. Plan B was to use a [USB Wi-Fi dongle](https://www.asus.com/ca-en/networking-iot-servers/adapters/all-series/usb-ac53-nano/)<sup>[4]</sup> instead. Now I know that this dongle works with Linux, as I used it earlier before. So uninstalled the Wi-Fi chip to ensure that it wouldn't be used in the laptop, and reinstalled everything back to their spots. After that, I was able to boot back into Windows, confirming that I was able to successfully disassemble and reassemble a laptop. This made me realize that it isn't that scary to take apart a laptop, and that let me unlock new options in the future.  

## Connecting to the Wi-Fi shouldn't be that hard, right?
After plugging in the USB dongle into the laptop, I installed Proxmox 9.1 to the laptop. But I quickly realize that I couldn't, since Proxmox expects the machine to be using Ethernet instead of Wi-Fi, like a normal server would be using. Too bad that isn't an option here. Now I could be using something else then Proxmox as my server, but I really like to use VMs for the home lab, and their web UI makes it easy to do.  Luckily for me, I know that there is a way to [install Proxmox on top of Debian 13](https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_13_Trixie)[5]. So I installed Debian 13 with the KDE's UI. After a painless install, I was able to connect the dongle to the Wi-Fi. Since the KDE's UI was only temporary to connect to the Wi-Fi, I uninstalled it to save on storage and ram usage (especially in this economy). However, I realize after I uninstall KDE that it also uninstalled `NetworkManager`. Which as the name suggests, is a tool to manage network connections. So now I just have Debian without any way to connect it to the internet.

As I reinstalled Debian 13, I realized that there are two paths I can take. The first one is the simple way where I installed Proxmox on top of Debian with KDE to connect it to the internet. The second option is to figure out how to connect to the Wi-Fi using the terminal. Considering that I just uninstalled KDE to save some system usage, I picked the second option. Now while I have connected to a Wi-Fi using the terminal before, I have never done it to a WPA2 enterprise network before. However, I used `NetworkManager` to do that, which Debian doesn't use. 

While I did attempt to learned how to use Debian's `ifupdown` tool, I quickly got confused on how to connect it to the enterprise network and gave up on it. So instead, I set up a hotspot on my phone, used `ifupdown` to connect it to my phone (which is easier to do), and installed `NetworkManager`. Using `NetworkManger`'s `nmtui` tool, I attempted to connect it to the network. However, `NetworkManager` isn't seeing the USB Wi-Fi dongle's interface. I was confused since it was able to see it earlier when I used KDE. Turns out, only one network inference's tool can manage the Wi-Fi interfaces at a time<sup>[6]</sup>, which `ifupdown` was hoarding them. So after disabling it and restarting `NetworkManager`, `nmtui` was able to successfully connect to the network. 

However, I realized something after I connected it to the network. Since Proxmox is built on top of Debian with only the `standard system utilities`<sup>[5]</sup>, it wouldn't be using `ifupdown` instead of `NetworkManger`. So I can't use `NetworkManager`. So after uninstalling `NetworkManager` and re-enabling `ifupdown`, I am back to square one. I then learned that I am suppose to use `wpa_supplicant` to connect it to the network<sup>[7]</sup>. So after reading the [Gentoo Wiki on wpa_supplicant](https://wiki.gentoo.org/wiki/Wpa_supplicant#WPA2_with_wpa_supplicant)<sup>[8]</sup>, I learned that I am suppose to create an entry in the `/etc/wpa_supplicant/wpa_supplicant.conf` file that contains the information to connect to the Wi-Fi. I also learned from the [Debian Wiki](https://wiki.debian.org/WiFi/HowToUse#A.2Fetc.2Fnetwork.2Finterfaces_and_wpa_supplicant)<sup>[9]</sup> that I connect that `wpa_supplicant.conf` file to the interface.  However, I wasn't able to get `wpa_supplicant` to connect to the Wi-Fi using the template that the Gentoo Wiki provided. I've tried many different templates to get it to connect that I can find online.  Eventually, I was able to connect to the network using a config that Google Gemini generated for me. 
```TOML
ctrl_interface=DIR=/run/wpa_supplicant GROUP=netdev
update_config=1

network={
        ssid="SSID"
        key_mgmt=WPA-EAP
        eap=PEAP
        identity="USERNAME"
        password="PASSWORD"
        phase2="auth=MSCHAPV2"
}
```

I then attach the `wpa_supplicant` config file to the `/etc/network/interfaces` so that the Wi-Fi USB dongle can use it.
```python
auto wlx08bfb84f0c66
iface wlx08bfb84f0c66 inet dhcp
        gateway 10.77.160.1
        wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

I then attempt to make the IP address static, or else the IP will change every so often, making the server annoying to use.
Luckily for me, it was easy to do, since the [Debian Wiki](https://wiki.debian.org/NetworkConfiguration#Configuring_the_interface_manually) has a section on it. Or at least it should have been easy to do, but the static IP wasn't working with me. I know it's a static IP issue because as soon as I switch back to DHCP, it was working just fine. I have tried many different guides and reboots, but none of them worked for me. I've even asked Google's Gemini to help me solve this issue. That's when Gemini suggested that the dorm's network could have been blocking static IPs from the network. That's when it all makes sense on why no one else's guide was working for me, as that is the only possible explanation. So I'm stuck with DHCP.   
## Forcing Proxmox to use Wi-Fi
After successfully connection Debian to the network, I have two options to go from here. I could either install Proxmox on top of Debian 13, or I install Proxmox and get the `wpa_supplicant` on there. Since apparently I'm a sucker for harder options, I went for the second option. The reason why is that I was worried that installing Proxmox on top of Debian would be missing something, or a feature would be misconfigured, causing future headaches. 

At this point, I've decided to do something smart, for once in my life, using VMs to test changes before doing it on the laptop. So I've installed Proxmox into a fresh QEMU VM to see if it comes with `wpa_supplicant` pre-installed. Spoiler alert, it doesn't. Which makes sense since Proxmox expects you to use Ethernet, not Wi-Fi.  So I would have to find someway of loading the `wpa_supplicant` package into Proxmox without internet. So I found a tool that lets me repackage Debian packages into a `.deb` file<sup>[9]</sup>. This is great as any Debian system can install `.deb`. The only catch is that I would need to install `dpkg-repack` in order to do that. So I created a Debian VM and repackage the `wpa_supplicant` file via `dpkg-repack wpasupplicant`. I then created another Debian VM to test if the package works. So after creating the VM, I plugged the USB dongle into my machine, passed it into the VM, installed the package via `dpkg -i wpasupplicant_2.10-24_amd.deb`, setup `wpa_supplicant` on the VM, and I was able to successfully connect the VM to the dorm Wi-Fi via the dongle. 

Went back to the Proxmox VM I've just installed, and installed the `wpa_supplicant` package to it. However, I was experiencing a weird issue where USB Dongle was randomly disconnecting from the Proxmox VM. The Debian VM was power off, so it can't be that. I was struggling find the answer online, and I got to the point where I decided to abandoned the VM and test it live on the laptop. After a fresh install of Proxmox, installing `wpa_supplicant`, and connecting `wpa_supplicant` to the network, the laptop couldn't connect to the network. At first, I thought Proxmox isn't compatible with `wpa_supplicant`, but then I notice something weird. The laptop has gotten an IP from the network, so it was able to successfully connect to the network. And I can confirm this by pinging the laptop. I then check the laptop's gateway and notice it was empty, which is funny since the gateway is, most likely, the one that gave it the IP info. After manually assigning the gateway, it was able to ping [google.com](https://www.google.com).  

## Fixing Proxmox's Network Issues. 
I was then able to visit the Proxmox Web UI, confirming that it's working. However, since the laptop is forced to use DHCP, I would have to manually configure the `/etc/hosts` since Proxmox requires that to be accurate<sup>[11]</sup>. Luckily for me, a user named groque had made a [bash script](https://forum.proxmox.com/threads/set-dhcp-on-proxmox.168358/#post-782918) that will automatically update that whenever it gets the DHCP info. However, I also wanted the script to update the Proxmox's splash screen, with the new IP, whenever the machines boots up.

![Pasted%20image%2020260315143827.png](Pasted%20image%2020260315143827.png)
*The splash screen whenever the machines boots up*

A quick google search shows that the splash screens lives in the `/etc/issue` file<sup>[13]</sup>. So I've modified the script from earlier to also update the splash screen by adding `sed -i "s/https:\/\/[0-9]\{1,3\}\(\.[0-9]\{1,3\}\)\{3\}/https:\/\/$new_ip_address/" /etc/issue`. Just want to clarify that I did use Google's Gemini to generate that `sed` command, since I still don't understand `sed`'s (and regex's) syntax. But the command still works after multiple reboots. 

Now the last thing that I had to do is to create a network bridge for the VMs to use. Since I had access to the WebUI, I used it to create a bridge for me. After creating a bridge, I needed to test the network to ensure that it works. So I created a VM with a Arch Linux ISO. The reason why is that it is the lightest distro I can think of that has a live mode for me to test the network with. After Arch booted up, I attempted to ping [google.com](https://www.google.com), 8.8.8.8, and my local machine, but all of them failed. That means something is wrong with the bridge. To save you some bit of time, the same thing happened with me attempting to assigned a static IP. Where I google why the bridge isn't working, use other people bridges as a template, give up on googling and went to Gemini, and Gemini suggested that the dorm's network has a firewall that is blocking the bridge from working. 

So because of that, I would have to setup a NAT in order for the VMs to have internet connection. I found a [guide online](https://cloudspinx.com/creating-private-network-bridge-on-proxmox-ve-with-nat/) that tells me on how to create a NAT interface<sup>[14]</sup>. Basically, what the guide does is that it creates a new interface for the NAT, then it tells the primary interface (the one that is connected to the network) that it should masquerade the NAT interface and IP forward the packets to it, and finally start it up in order to be able to use it. After I follow the guide, the Arch VM is able to ping [google.com](https://www.google.com).

The very last thing I would need to do is to setup port forwarding so I can access the services from the VM. At this point, I just wanted to get this laptop setup, as it has taken me 3 days to get to this point. So I've asked Gemini on how to setup port forwarding. But before I can port forward, I have to have a service to test it on. So I've created an Ubuntu 24.04 container as the base. I then asked Gemini for how I can create a very basic website for this test, and it told me to run `python3 -m http.server 90`, which will create a web server that contains all of the files in the current directories (after I make some). Then I asked it how I can port forward on the NAT interface, and it gave me the following `iptables` lines to insert into the NAT interface.
```bash
post-up   iptables -t nat -A PREROUTING -i wlx08bfb84f0c66 -p tcp --dport 8080 -j DNAT --to-destination 192.168.50.2:90
post-down iptables -t nat -D PREROUTING -i wlx08bfb84f0c66 -p tcp --dport 8080 -j DNAT --to-destination 192.168.50.2:90
```
To quickly explain the commands, the `post-up` & `post-down` are the `iptables` commands to run when the interfaces goes up or down. The `-t nat` tells `iptables` that the command is for a NAT table. The `-A` tells `iptables`that it should add this rule, while the `-D` tells `iptables to delete the rule`. The `-i wlx08bfb84f0c66` is the name of the interface to port forward from (`wlx08bfb84f0c66` is the interface of the USB dongle). The `-p tcp` tells `iptables` that the port is a TCP port. the `--dport 8080` tells `iptables` that the incoming port should be set to 8080. The `-j DNAT` tells `iptables` that the destination should be inside the NAT.  And finally, the `--to-destination 192.168.50.2:90` tells `iptables` what machine and port the service is at.

## Don't go to sleep, Proxmox.
The laptop is almost ready to be used. I want the server to continue to run after I close the lid. Since Debian uses Systemd, I can modify the `logind.conf` file to tell it how to handle laptop lids<sup>15</sup>. Specially, I set the `HandleLidSwitch`, `HandleLidSwitchDocked`, and `HandleLidSwitchExternalPower` to all be ignore. Then I have to restart the `systemd-logind` service (`systemctl restart systemd-logind`) in order for it to read the changes. Now when I close the lid, the laptop doesn't suspend. 

Now this should be fine, except there is one problem with this. When the lid closes, it cuts all power to the USB ports, which is where the Wi-Fi is living at. So I am back at square one. From what I've seen from search online, the only option is to enable that feature in the BIOS, as the BIOS settings will take higher priority over the kernel. Unfortunately for me, the BIOS is barebones. So my options are either to update the BIOS, replace the Wi-Fi dongle with an actual Wi-Fi chip, or just stop here. Now Updating the BIOS can be risky, since losing power during the process can brick the motherboard. And with the recent weather problem causing me to lose power, this isn't an option for me. Especially since I'm not even sure if the most up to date BIOS has that option. And at this point, I'm too lazy/impatient to replace the Wi-Fi chip. So I'm going to have to live with a open laptop screen, which isn't that big of a deal.  Especially since I can stick some object inside the laptop, prevent the lid from fully closing. This allows me to place the laptop in a 45 degree on the ground. 

## Give me some space back. 
Now I forgot about this when I installed Proxmox, but it splits the storage into two storage pools, local and local-lvm. Normally this would be fine, as there will be enough disk space for both of them. However, since it didn't do that for me. 
![Pasted%20image%2020260319092713.png](Pasted%20image%2020260319092713.png)
*The info the the local storage pool*
![Pasted%20image%2020260319092728.png](Pasted%20image%2020260319092728.png)
*The info for the local-lvm storage pool*

So because of that, I want to combine both of them into one pool. Now since this might be risky, and prevent the system from storing things properly, I will be testing it in the Proxmox VM. I then followed the [guide I found online](https://blog.gnomeitsolutions.com/delete-local-lvm-proxmox/) that tells me how delete the lvm, and give that storage to the local storage<sup>[16]</sup>. But basically it goes how you expect it to go, you delete the storage in Proxmox, make the local storage store everything, purge the lvm storage with `lvremove /dev/pve/data`, assign the local storage the lvm storage size via `lvresize -l +100%FREE /dev/pve/root`, and resize the local storage to use the new storage space via `resize2fs /dev/mapper/pve-root`. After that, I then testing if I can storage disk images and containers in the local storage, and if backup/snapshots still work, and they do. So then I actually run this on the actual laptop, and I had no issues with it.  

## Ethernet for the win.
Since having the server's IP constantly change would be annoying, I've grabbed a small switch and two Ethernet cables to fix that. With the ethernet cables, I can assign a static IP to the ethernet cables, and connect to the web UI via that. The Wi-Fi chip will still be used to connect the machine, and VMs/LXCs, to the internet. I've reused the existing network in the `vmbr0` in Proxmox, and setup my device's ethernet to use that network. With that, I can successfully connect to the web UI.   

## Routing issue
So ever since I've plugged in the Ethernet cable, I've notice that the Wi-Fi chip was unable to ping the internet. At first, I thought something went wrong with the `/etc/network/interface` file, so I've spent some time trying to "fix" it. It, only when I ran `ip route` that I saw the issue:
```bash
default via 192.168.100.1 dev vmbr0 proto kernel onlink 
10.77.160.0/19 dev wlx08bfb84f0c66 proto kernel scope link src 10.77.160.104 
192.168.50.0/24 dev vmbr1 proto kernel scope link src 192.168.50.1 
192.168.100.0/24 dev vmbr0 proto kernel scope link src 192.168.100.2
```

The default route for accessing outside the network was using the Ethernet port, not the Wi-Fi chip. My guess is that when I plugged the Ehternet port, Proxmox must have modified the default route.  Sure enough, when I've deleted the default route via `ip route del 0.0.0.0/0`, and added a new default route through the Wi-Fi chip via `ip route add 0.0.0.0/0 via 10.77.160.1 dev wlx08bfb84f0c66`, I am now able to ping [google.com](https://www.google.com).

However, when I reboot the laptop, it goes back to the old route. To prevent that, I've modified the `/etc/network/interface` file to override that by adding the following to the `vmbr0` interface.
```toml
post-up ip route delete 0.0.0.0/0 
post-up ip route add 0.0.0.0/0 via 10.77.160.1 dev wlx08bfb84f0c66

post-down ip route delete 0.0.0.0/0 
post-down ip route add 0.0.0.0/0 via 192.168.100.1 dev vmbr0
```
These commands do two parts. When the `vmbr0` interface goes up, it will delete the existing default route via `post-up ip route delete 0.0.0.0/0`. 
Then, it will create a new default route (`0.0.0.0/0`) to the router (`10.77.160.1`) via the Wi-Fi chip (`wlx08bfb84f0c66`). Then, when the interface goes down, it will delete the new default route (`post-down ip route delete 0.0.0.0/0`), and re add in the old default route to the ethernet port (`ip route add 0.0.0.0/0 via 192.168.100.1 dev vmbr0`)

## Citations
>  [1] “ASUS X555Q 15.6" Laptop AMD A12-9720P - Newegg.com.” _Newegg.com_, 2021, www.newegg.com/global/ph-en/asus-15-6-non-touch-screen-2-70ghz-8gb-memory-128-gb-ssd-black/p/1TS-001A-00G25. Accessed 7 Mar. 2026.
>  [2] CFi Computer. “ASUS X555Q SSD & RAM Upgrade | Disassembly and Cleaning Tutorial!” _YouTube_, 28 Apr. 2025, www.youtube.com/watch?v=kpkZ6dwhnB0. Accessed 7 Mar. 2026.
>  [3] Amazon. “MPE-AXE3000H Mini PCIE Wifi Card WiFi 6E AX210 AX3000Mbps Wifi Bluetooth Adapter .” _Amazon.com_, dreeong, 29 Nov. 2024, www.amazon.ca/MPE-AXE3000H-AX210-AX3000Mbps-Bluetooth-Adapter/dp/B0DP9YKVRL. Accessed 7 Mar. 2026.
>  [4] “USB-AC53 Nano.” _ASUS Canada_, 2025, www.asus.com/ca-en/networking-iot-servers/adapters/all-series/usb-ac53-nano/. Accessed 7 Mar. 2026.
>  [5] “Install Proxmox ve on Debian 13 Trixie - Proxmox VE.” _Proxmox.com_, 2025, pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_13_Trixie. Accessed 7 Mar. 2026.
>  [6] “NetworkManager Refuses to Manage My WLAN Interface.” _Ask Ubuntu_, 3 Nov. 2012, askubuntu.com/a/820513. Accessed 7 Mar. 2026.
>  [7]  ANeilan. “How to Connect to University Wpa2 Enterprise Network on Debian? .” _Reddit.com_, 15 Aug. 2015, www.reddit.com/r/linux/comments/3ghgmm/comment/cty5uw5/. Accessed 7 Mar. 2026.
>  [8] “Wpa_supplicant - Gentoo Wiki.” _Gentoo.org_, 2025, wiki.gentoo.org/wiki/Wpa_supplicant#WPA2_with_wpa_supplicant. Accessed 7 Mar. 2026.
>  [9] “NetworkConfiguration - Debian Wiki.” _Debian.org_, 26 Nov. 2025, wiki.debian.org/NetworkConfiguration#Configuring_the_interface_manually. Accessed 7 Mar. 2026.
>  [10] Panta. “How to Create a .Deb File from Installed Package?” _Ask Ubuntu_, 4 Mar. 2015, askubuntu.com/a/592553. Accessed 7 Mar. 2026.
>  [11] Hannes Laimer. “Set DHCP on Proxmox.” _Proxmox Support Forum_, 11 July 2025, forum.proxmox.com/threads/set-dhcp-on-proxmox.168358/#post-782909. Accessed 7 Mar. 2026.
>  [12] groque. “Set DHCP on Proxmox.” _Proxmox Support Forum_, 11 July 2025, forum.proxmox.com/threads/set-dhcp-on-proxmox.168358/#post-782918. Accessed 7 Mar. 2026.
>  [13] DvM. “Wrong Welcome Message after Changing IP.” _Proxmox Support Forum_, 8 Feb. 2018, forum.proxmox.com/threads/wrong-welcome-message-after-changing-ip.41290/#post-198754. Accessed 7 Mar. 2026.
>  [14] Mutai, Josphat . “Creating Private Network Bridge on Proxmox ve with NAT - CloudSpinx.” _CloudSpinx: Empowering Businesses in the Cloud_, CloudSpinx, 4 Mar. 2025, cloudspinx.com/creating-private-network-bridge-on-proxmox-ve-with-nat/. Accessed 7 Mar. 2026.
>  [15] “Power Management - ArchWiki.” _Archlinux.org_, 2023, wiki.archlinux.org/title/Power_management#ACPI_events. Accessed 7 Mar. 2026.
>  [16] dinesh. “How to Delete Local-LVM and Resize Storage in Proxmox ve - Gnome IT Solutions | Web Hosting, VPS & Server Tutorials.” _Gnome IT Solutions | Web Hosting, VPS & Server Tutorials_, 28 June 2023, blog.gnomeitsolutions.com/delete-local-lvm-proxmox/. Accessed 7 Mar. 2026.
