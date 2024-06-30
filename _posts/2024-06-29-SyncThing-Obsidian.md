---
title: "SyncThing to sync my Obsidian folders across different platforms"
layout: single
tags:
  - docker
  - Sync
  - SyncThing
  - Obsidian
  - Compose
  - backup
toc: true
toc_label: "Outline"
toc_icon: "fa fa-spinner fa-spin"
toc_sticky: True
description: I started to use Obsidian as an PKM (Personal Knowledge Management) - needed a way to sync data across multiple platform - Android, Apple Mac and Linux. Rather not keep my private obsidian notes in the cloud storage - like box, google drive etc..
categories: posts
sitemap: true
pkeywords: Sync, docker, Compose, obsidian, 
---
I recently stumbled across a book called "Building a second brain" by Tiago Forte. The book talks about the use of PKM (Personal Knowledge management) as a Second Brain to manage the knowledge and facts that we have acquired through our lives. This frees up the biological brain to be more creative and be more present.

Obsidian is the PKM tool I am testing out. Obsidian stores all the notes inside a Vault - a vault is essentially is one main folder that contains multiple sub-folders, mark down files, and image files etc. Obsidian allows you to leaverage Public Cloud infrastructure such as iCloud and Obsidian Sync (Paid Service) to store and sync your vault, but I like my private notes to only exists on my private infrastructure.

In this blog, I will discuss how it is possible for the same Vault to be available across multiple different type of devices like my Samsung Phone, Apple MacBook and my NAS. With it being so available, I will have access to my private notes across my phone and laptop. The copy of the vault on the NAS is more for redundancy purposes. If I lose my phone and laptop at the same time, I will still have a copy of the Vault on my NAS.

***
### Requirements

Here are my requirements for syncing my vault:
* Need to access to the same notes (Vault) from any of my devices (Linux,Apple Mac and Android).
* Any changes in the note on any devices should be available on other devices - subject to network being available.
* If there is no network available when changes are made, those changes should be propagagated when the network is available.
* If I am outside my home, any changes on my devices should propagate back to my NAS via my Linux Server. My NAS is not running a useful version of Linux.
* Simple file versions, I want to be able recover any file I accidentally deleted or changed.

Based on the above requirements I have implemented [SyncThing](https://syncthing.net/). An Opensource solution that is supported across multiple platforms.

***
### Use Cases
The diagram below illustrates two important Use Cases for me:
#### - At home
If I am at home, I might want to work on my Obsidian notes that is on my laptop. Any changes on my laptop should be synced across to the other devices, including the Linux Server(NAS) and phone (if unlocked). If the phone is locked, it will sync the next time the phone is unlocked.
Similiarly, any changes on the Phone will be sync to the Laptop (if turned on) and NAS. The devices will sync automatically when they come online from either the Linux Server(NAS) which is always on or the other devices.

#### - On the train
On the train, I usually use my mobile as a hot spot when I work on the laptop. Any changes on my laptop will be propagated to my phone and to my NAS at home. But if I cannot get a seat on the train, I could still update my obsidian notes on my phone and any changes will be propagated to the NAS at home, and when the laptop comes online it will be sync with an updated copy.

The train use case could be easily extended to anywhere outside of the home, like the work place, public library etc..

[![](/assets/images/2024-06-29-SyncThing_Infra.gif)](/assets/images/2024-06-29-SyncThing_Infra.gif)

***
### Components used
* On the Linux Server - mount a volume on to the NAS and make it available for the docker container.
* On the Linux Server - spin up a SyncThing docker container based on the Docker-compose YAML file specified below.
* Install SyncThing for laptop from [SyncThings Download](https://syncthing.net/downloads/)
* Install SyncThing for Android device from Google Play or F-Droid.

<i class="fas fa-regular fa-star fa-2x fa-spin"></i> 
Note: Ideally run the docker instance in Host Network Mode as this exposes the ports directly to the host rather than through a NAT'ed docker network.

***
### Code break down
> Analysis of Docker-Compose.yml file


{% highlight yaml linenos %}
---
services:
  syncthing:
    image: syncthing/syncthing
    container_name: syncthing
    hostname: syncthing
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Australia/Sydney
    volumes:
      - /home/drone/docker/syncthing:/var/syncthing
      - /mnt/SyncThing:/data
    network_mode: host
    restart: unless-stopped
{% endhighlight %}

*Line 8 and 9* - allows our containers to map the container's internal user to a local user on the host machine. On the host machine I use an user called 'drone'. Do the following to find out the id for the user(UID) and the group(GID) for your file.

{% highlight bash linenos %}
> id drone
uid=1000(drone) gid=1000(drone) groups=1000(drone),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),999(docker)
{% endhighlight %}

*Line 12* - maps a folder on the host /home/drone/docker/syncthing to the directory inside the container /var/syncthing - this is where all the SyncThing configurations are stored.

*Line 13* - I have create a network mount in /mnt/SyncThing to my NAS and this is mapped internally to the docker container /data.

*Line 14* - Container will use 'host' network mode - this will avoid the Global Discovery and local discovery issues.

In the same folder as where your docker compose file sits, issue 'docker compose up -d' to install and start the container.
Once its status is "healthy", you can browse to it on port 8384. http://x.x.x.x:8384

{% highlight bash linenos %}
> docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED          STATUS                    PORTS                                                                                  NAMES
d8d0e7d021b6   syncthing/syncthing     "/bin/entrypoint.sh …"   14 minutes ago   Up 14 minutes (healthy)                                                                                          syncthing
{% endhighlight %}

### Things to be aware of and minor tweaks

* The SyncThing for Linux and Mac Book both needs to have a local GUI username and password set.

[![](/assets/images/2024-06-29-Warning.png)](/assets/images/2024-06-29-Warning.png)

* Change the web GUI to use HTTPS instead of HTTP - in the drop down menu [Actions -> GUI]
* For my Obsidian folder, I use "Simple File Versioning" for [File Versioning](https://docs.syncthing.net/v1.27.7/users/versioning)
* For Android - I would get frequent "sync conflict", on the android device I had set "Max Conflicts = 0" for the obsidian folder. To access this setting, Web GUI -> Advanced -> Folders -> "my obsidian folder" -> Max Conflicts.


### SyncThing Configuration mapping 
Below is a configuration layout of my Linux Server (Mikasa) and Intel Mac Book. Also I have two android devices. All these devices would sync to and from each other.

| Configuration on | Folder label | Folder Path | Sync to Devices |
| :--- | :--- | :--- | :--- |
| Mikasa | obsidian-sync | /data/Sync_obsidian | Intel_MacBook, S9_Tablet, SamsungS22 |
| Intel_MacBook | obsidian-sync | /Users/pnhan/Documents/Obsidian Vault | Mikasa, S9_Tablet, SamsungS22 |
| S9_Tablet | obsidian-sync | /storage/emulated/0/Sync_Obsidian | Mikasa, Intel_Macbook, SamsungS22 |
| SamsungS22 | obsidian-sync | /storage/emulated/0/Sync_Obsidian | Mikasa, Intel_Macbook, S9_Tablet |

<i class="fas fa-regular fa-star fa-2x fa-spin"></i> 
Please Note: Images below has some sections pixelated deliberately

[![](/assets/images/2024-06-29-MacBook.png)](/assets/images/2024-06-29-MacBook.png)

[![](/assets/images/2024-06-29-Mikasa.png)](/assets/images/2024-06-29-Mikasa.png)

### Screenshots from Train Use Case.
In the Train Use Case - the mobile phone is providing wifi Hot Spot coverage for the laptop.

Below is a screen shot from SyncThing on the phone of the laptop. Note the IP address and the Connection type.
* IP address is from DHCP subnet that the phone provides to the laptop, while on the tethered Wifi connection.
* Connection Type is using TCP LAN to sync between phone and Laptop.

[![](/assets/images/2024-06-29-Mobile_TetherViewOfLaptop.png){: width="300" }](/assets/images/2024-06-29-Mobile_TetherViewOfLaptop.png)

Below is a screen shot from SyncThing on the Laptop of the phone.
* IP address is the phone IP address on the tethered Wifi connection.
* Connection Type is using TCP LAN to sync between phone and Laptop.

[![](/assets/images/2024-06-29-Laptop_Mobile.png)](/assets/images/2024-06-29-Laptop_Mobile.png)

Below is a screen shot from SyncThing on the Laptop of Mikasa (Linux Server).
* To reach Mikasa (Linux Server) - the IP address is picked up from the Global discovery Service that SyncThing provides.
* Connection Type is using QUIC WAN to sync between Mikasa (Linxu Server) and Laptop. To reach Mikasa, the laptop would need to NAT out the phone's Hot Spot, and traverse over the Internet to a dynamically assigned IP address that is provided by my Service Provider.

[![](/assets/images/2024-06-29-Laptop_Mikasa.png)](/assets/images/2024-06-29-Laptop_Mikasa.png)

***

### Summary
SyncThing is quite powerful and yet it is relatively straight forward to setup. I have also used it to overcome sharing files between Mac Book and Android, since there is no "Air Drop" equivalent for Android and Apple Mac.
Anyway have fun playing with it.

Please reach out if you have any questions or comments.<br>

