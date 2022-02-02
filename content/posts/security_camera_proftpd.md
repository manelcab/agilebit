---
title: FTP Daemon on Docker Raspberry PI
subtitle: How to use proftp on docker in a raspberry PI and receive images/videos from your security camera
category:
  - PI
author: lenambac
date: 2022-01-30T18:23:56.800Z
featureImage: /uploads/proftp.png
---

Hi all, in this post we are going to run [Proftpd](http://www.proftpd.org/) on docker and we'll configure a list of users authorized to use it.

If your security camera can be configured to send images/videos to a remote ftpserver, then we can use our raspberry to receive and save these files.

This is the [git repo](https://github.com/kibatic/docker-proftpd) of the proftd server we'll use

These are the steps:

## 1. Create user files

First of all create a new file and include a live with each user you want to configure, where the syntax is username:password, like this:

```
cat /home/pi/config/ftpd/users.conf
username:password
``` 

## 2. Start PROFTP Daemon

It is recommend that you use an usb pen drive to reduce the number of IOs of the raspberry SD card.
In our case  our usb pen drive is mounted in /mnt/usb0/proftp.

Create first this directory with 777 permision in order to see which id is used by proftpd daemon. 
```
sudo chmod 777 /mnt/usb0/proftp
```

Start with the following command:

```
users=`cat /home/pi/config/ftpd/users.conf` ; docker run -d --restart=unless-stopped --net host -e FTP_LIST=$users -e USERADD_OPTIONS="-o --gid 33 --uid 33" -v /mnt/usb0/proftp:/home/camera rpi-proftpd

``` 
Check the user and group id and modify the destination directory (/mnt/usb0/proftp):

```
sudo chown id:group /mnt/usb0/proftp
sudo chmod 755 /mnt/usb0/proftp
```


## 3. Configure your security camera

Configure your security camera with the raspberry ip address, username and password.


## 4. House keeping

If you want to limit the number of files saved on your local host, you can use the following container to delete all the files every 'n' months.

The local directory /mnt/usb0/proftp/ is mounted and deleted all files every 15 days (1296000 seconds)

```
docker run -d -v /mnt/usb0/proftp/:/mnt/ftpd --restart=unless-stopped alpine /bin/ash -c 'while : ; do df -h /mnt/ftpd/ >> /mnt/ftpd/house_keeping.log ;  echo purging >> /mnt/ftpd/house_keeping.log ; rm -rf /mnt/ftpd/202* ; df -h /mnt/ftpd/ >> /mnt/ftpd/house_keeping.log ; sleep 1296000; done'
```

A next version of this command it could be a deletion of only files older than 1 month, 2 months, ...