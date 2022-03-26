---
title: Pi-hole on raspberry PI docker container
subtitle: How to use Grafana, Influx, Mosquito on Docker
category:
  - PI
author: lenambac
date: 2022-03-26T18:50:56.800Z
featureImage: /uploads/pihole.png
---

Hi all, in this post we are going to install [Pi-hole](https://pi-hole.net/) on Raspberry PI.

Pi-hole is a Linux network-level advertisement and Internet tracker blocking application which acts as a DNS sinkhole and optionally a DHCP server,


These are the steps:

## 1. Install docker-compose

First of all you need to install docker-compose:

```
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install libffi-dev libssl-dev
sudo apt install python3-dev
sudo apt-get install -y python3 python3-pip
sudo pip3 install docker-compose

```

## 2. Clone Pi-hole git repo and setup

Clone the repository:
``` 
git clone https://github.com/pi-hole/docker-pi-hole

```

Then update TZ variable with the correct value. It's better no indicate the webpassword on the configuration file, after the container will be running we can modify it.

In this confinguration file there are some volumes, the two first are related to pi-hole configuration and the last two are needed to avoid to much i/o on raspberry sd-card.

> note: the lighttpd volume is needed to avoid the following error: "cannot touch '/var/log/lighttpd/access.log': No such file or directory"

docker-compose.yaml config file:

```
# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    # For DHCP it is recommended to remove these ports and instead add: network_mode: "host"
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp" # Only required if you are using Pi-hole as your DHCP server
      - "80:80/tcp"
    environment:
      #TZ: 'America/Chicago'
      # WEBPASSWORD: 'set a secure password here or it will be random'
    # Volumes store your data between container upgrades
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'    
      - '/mnt/ram-pihole:/var/log'
      - '/mnt/ram-pihole/lighttpd:/var/log/lighttpd'
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    cap_add:
      - NET_ADMIN # Recommended but not required (DHCP needs NET_ADMIN)      
    restart: unless-stopped
```


## 3. Create a tmpfs RAM volumen to avoid writes on raspberry SD card

Include the following line in /etc/fstab
```
tmpfs   /mnt/ram-pihole    tmpfs   defaults,noatime,nosuid,size=64m,mode=755,user
```

Then create the directory and change the permisions:

```
sudo mkdir /mnt/ram-pihole
sudo chown pi:pi /mnt/ram-pihole
mount /mnt/ram-pihole
```

## 4. Start pi-hole

Start the pi-hole by docker-compose

```
docker-compose up -d
``` 


## 5. Change webpassword

Execute the following command to chage the password:

```
sudo docker exec -it pihole pihole -a -p
```

## 6. Access to pi-hole web interface

In your browser access with the raspberry ip address like:

http://<raspberry_ip_adress>/admin/

![pi-hole dashboard](/uploads/pihole-dashboard.png)



## 7. Update wifi configuration of your router

Firstly, it is recommended that you only modify the primary DNS server of one device in order to verify that all dns querIEs are running ok and advertisement are blocked.

After that, you can update the wify settings of your home router to configure the primary DNS server as pi-hole.
