---
title: Raspberry PI with Grafana Influx and Mosquitto on Docker
subtitle: How to use Grafana, Influx, Mosquito on Docker
category:
  - PI
author: lenambac
date: 2022-01-29T17:49:56.800Z
featureImage: /uploads/docker_grafana_influx.png
---

Hi all, in this post we are going to run the following components via [Docker](https://www.docker.com/) on Raspberry PI:
- [Grafana](https://grafana.com/)
- [Influx](https://www.influxdata.com/)
- [Mosquitto MQTT broker](https://mosquitto.org/)
- [Telegraf](https://www.influxdata.com/time-series-platform/telegraf/)
- [Proftpd](http://www.proftpd.org/)

These are the steps:

## 1. Raspberry Pi OS installation

First of all you need to [install the OS](https://www.raspberrypi.com/documentation/computers/getting-started.html) in the raspberry PI, there are several options you can use: Raspberry PI OS, Ubuntu, NOOBS, OSMC, OpenELEC ... , we'll use Raspberry PI OS (previopusly called Raspbian)

Follow the instrucctions on https://www.raspberrypi.com/software/ to install the Operating system.

### Static IP configuration

Because raspberry will be used as a server which will run several services, you can configure a static IP adress to make easy the configuration of all devices which needs to connect with raspberry.

If your local network is 192.168.1/24 and your default gateway is 192.168.1.1

Edit /etc/dhcpcd.conf file and modify the following lines:
```
# Example static IP configuration:
interface eth0
static ip_address=192.168.1.{IP_number}/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8
noipv6
``` 
Update the {IP_number} with a number of your choice between 2 and 254.
It is recommended that you configure your DHCP Server (usually on your router) in order to indicate that this IP adress is static and avoid IP duplication errors on your network. 



## 2. Docker installation

After the basic raspberry PI configuration is done we are going to install Docker.
Please see the [reference](https://phoenixnap.com/kb/docker-on-raspberry-pi)

```console
sudo apt-get update && sudo apt-get upgrade
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

Add a non_root user to the Docker Group

```console
sudo usermod -aG docker [user_name]
sudo usermod -aG docker Pi
```
Check Docker Version
```console
docker version
```
Run Hellow World
```console
docker run hello-world
```


## 3. Clone your Git reposity
If you have a gitlab repository with all your code, scripts, ... clone it in order to use them
Install git client and configure your [personal token](https://docs.github.com/es/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token).

```
git config --global user.name "manelcab"
git config --global user.email "manelcab@gmail.com"
```

If you want to avoid entering always the token you can follow these steps, but remember that this token will not be encrypted on you system

```
git config --global credential.helper store
cat ~/.git-credentials

``` 

## 4. Copy your data
After clonning your repository you can copy your data and secrets, for exempla on /home/pi/config
An easy way is to use a USB drive where you can place all you data and config. Remember to do backup copies!.

Check the devices with:
```
lsblk -fp
fdisk -l
```

Create a new partition type 83 (Linux)
```
fdisk /dev/sda
sudo mkfs.ext4 /dev/sda1
``` 
Copy the PARTUUID

```
sudo blkid
```
Include the following line int /etc/fstab with the correct PARTUUID

```
PARTUUID=XXXXXXXX-XX /mnt/usb0 ext4 defaults,auto,users,rw,nofail 0 0

```

## 5. Starting mosquitto

If it is the first time and you don't have the mosquitto.conf in your config directory, then you can create a temporary container and copy the file from the image to your host:

```
id=$(docker create eclipse-mosquitto)
docker cp $id:/mosquitto/config/mosquitto.conf /home/pi/config
docker rm -v $id
``` 

Now you can modify the mosquitto parameters. After than start mqtt broker with the following command. If you specify unless-stopped parameter this docker will start again after reboot.


```
docker run -d -p 1883:1883 -p 9001:9001 --restart=unless-stopped -v /home/pi/config/mosquitto/mosquitto.conf:/mosquitto/config/mosquitto.conf eclipse-mosquitto
``` 

## 6. Starting influx

Start influx with:

```
PWD=/home/pi/config/influxdb; docker run -d -p 8086:8086 --net=host --restart=unless-stopped -v $PWD:/var/lib/influxdb arm32v7/influxdb
```

Now we have to create a new database for the topics of mosquitto mqtt broker, so launch influx client with the following command:

```
id=$(docker ps -q --filter ancestor=arm32v7/influxdb); docker exec -ti $id influx
```

Here you can see that there is only one database

```
Connected to http://localhost:8086 version 1.8.4
InfluxDB shell version: 1.8.4
> show databases
name: databases
name
----
_internal
```

We can now create a new database and user admin, choose your password:

```
CREATE DATABASE topics
CREATE USER admin WITH PASSWORD 'xxxx' WITH ALL PRIVILEGES
GRANT ALL PRIVILEGES ON topics to admin
auth (admin/xxxx)
use topics
show measurements
quit
```


## 7. Starting telegraf

If it is the first time and you don't have the telegraf.conf in your config directory, then you can create a temporary container and copy the file from the image to your host:

```
id=$(docker create arm32v7/telegraf)
docker cp $id:/etc/telegraf/telegraf.conf /home/pi/config
docker rm -v $id
``` 

Then modify your telegraf.conf local copy with the name of the influxdb created previously:

```
# Configuration for sending metrics to InfluxDB
[[outputs.influxdb]]
  ## The full HTTP or UDP URL for your InfluxDB instance.
  ##
  ## Multiple URLs can be specified for a single cluster, only ONE of the
  ## urls will be written to each interval.
  # urls = ["unix:///var/run/influxdb.sock"]
  # urls = ["udp://127.0.0.1:8089"]
  # urls = ["http://127.0.0.1:8086"]

  ## The target database for metrics; will be created as needed.
  ## For UDP url endpoint database needs to be configured on server side.
  # database = "telegraf"
   database = "topics"
   username = "admin"
   password = "xxxxx"

```

Then configure the mosquitto parameters and include all your topics you want to read from mosquitto broker:

```
# # Read metrics from MQTT topic(s)
 [[inputs.mqtt_consumer]]
#   ## MQTT broker URLs to be used. The format should be scheme://host:port,
#   ## schema can be tcp, ssl, or ws.
#   servers = ["tcp://127.0.0.1:1883"]
#
#   ## Topics that will be subscribed to.
#   topics = [
#     "telegraf/host01/cpu",
#     "telegraf/+/mem",
#     "sensors/#",
#   ]
  #servers = ["tcp://localhost:1883"]
  servers = ["tcp://{YOUR_RASPBERRY_IP_ADRESS}:1883"]
  qos = 0
  connection_timeout = "30s"


  topics = [
    "telegraf/host01/cpu",
    "telegraf/+/mem",
    "{all_other_topics}",
  ]

```

Start telegraf with the following command

```
docker run -d --net=host --restart=unless-stopped -v /home/pi/config/telegraf.conf:/etc/telegraf/telegraf.conf:ro --name telegraf arm32v7/telegraf
```


## 8. Starting Grafana

Create your config directory for grafana and change the user and group id to avoid permissions errors:

```
mkdir /home/pi/config/grafana
sudo chown 472:472 /home/pi/config/grafana
```

Start grafana with the following command

```
docker run   -d --net=host --restart=unless-stopped --name grafana --mount type=bind,source="/home/pi/config/grafana",target=/var/lib/grafana grafana/grafana-arm32v7-linux
```

Go to http://{Rasperry_PI_IP}:3000/ and enter admin for username and password (change the password after).

Create a new datasource with the follwing data:
- url: http://localhost:8086
- DB: topics (enter username and password)


## 9. Create a Grafana Dashboard

After some minutes the messages received by the mqtt broker topics will be gathered by telegraf and inserted in Influx database.

Create a new dashboard and add a new pannel, in the following image you can find and example of cpu_idle and cpu_wait

![Grafana_panel](/uploads/grafana_panel.png)

Then you can see a similar image like this:

![Grafana_panel](/uploads/grafana_cpu.png)


In case you have some devices which send messages to your mqtt broker, you can create panels with your data like this:


![Grafana_panel](/uploads/grafana_dashboard.png)






