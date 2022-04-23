---
layout: post
title:  "Draft: Everythink"
date:   2022-04-19 11:00:07 +0700
categories: everythink
---

## Docker MACVLAN on Synology NAS DSM

- [free your synology port & ip for docker containers](https://tonylawrence.com/posts/unix/synology/free-your-synology-ports/)

Free your Synology ports for Docker

I’ve been running Pi-Hole on my Synology for a good few years. It has taken me a while to figure out how to run it the way I liked to which is why I wrote the previous guide a few years ago. (see: Running Pi-Hole inside Docker on Synology)

Although this has helped me and many others, I was never quite happy about the outcome and have strived to find a better way. I didn’t want to have to rely on WebStation or anything else outside of the docker container. Thankfully I stumbled upon dockers network driver named macvlan.

    Note: I will be using the command line in this guide however the containers will still be visible in the Synology Docker UI.

TL;DR (too long, don’t read)

Download the docker-compose.yaml1 file to your Synology, edit all the network addresses replacing the example network 192.168.123.X with your own network. and run the following command:

```
$sudo docker-compose up -d
```

If it all works Pi-Hole will be running on IP .199. Continue reading to understand (hopefully!)

    If your names is using IP x.x.x.199 then please change the above!

Using macvlan for networking

Macvlan is a network driver provided by Docker, the following is an extract from the documentation

    Some applications, especially legacy applications or applications which monitor network traffic, expect to be directly connected to the physical network. In this type of situation, you can use the macvlan network driver to assign a MAC address to each container’s virtual network interface, making it appear to be a physical network interface directly connected to the physical network.

So the idea is that we will create our own docker network using the macvlan driver, this will then allow us to connect our Pi-Hole container onto this network which will assign it it’s own MAC. This will then appear to be directly connected on our host network but will have it’s own IP and therefore all network ports available.

A new network can be easily created using the following command (but hold off, we’ll go into actually doing this later using configuration files)

In all my examples I will be using the network 192.168.123.0/24. If you use any of the downloaded files please update these entries to your specific network.

```
$sudo docker network create 
    —driver=macvlan 
    —gateway=192.168.123.1
    —subnet=192.168.123.0/24 
    —ip-range=192.168.123.192/28 
    -o parent=eth0
    name-of-network
```

    Note: The IP address range in the above command of 192.168.123.192 / 28 might appear strange. This is so that we can restrict the IP addresses docker uses so not to clash with our physical machines see IP calculator.

Making things easier using docker compose

I’m quite a lazy guy when if comes to repitition. I quickly became fed up of clicking around the docker UI to create containers, update containers and to modify them. Luckily docker comes with a way to automate this setup with the command docker-compose. I now use this to create all my containers that I run but in this example we will focus on Pi-Hole.

    Note: I’m using version 2 of the docker compose format. This is the only version that seems to support specifying MAC and IP addresses which to me is very handy. It only needs to be specified once at the top of each file.

Docker compose requires a configuration file that is in YAML format. This is just plain text so can be edited using any application however, whitespace is important so no TAB characters please.

In the docker compose file we can create the network (macvlan), we can create our service (Pi-Hole) and optionally assign specific MAC and IP addresses. I will explain each section individually to build up the complete picture. I will also attach a download link at the bottom of the article so that you are not forced to copy & paste.

If you want to follow along then ssh to your Synology, create a file name docker-compose.yaml and add the following snippets. You can then try this out running the following command in the same place as the file you created. This command will re-create the config each time so you do not need to delete previous versions.

```
$sudo docker-compose up
```

    For more information try sudo docker-compose up —help

Start by specifying the version.

version: “2”                              # Version 2, only needed at top of each file

Defining the network:

```
networks:
  pihole_network:                         # Name of network
    driver: macvlan                       # Use the macvlan network driver
    driver_opts:
      parent: ovs_eth0                    # If open vSwitch is disabled use eth0 (or eth1 +)
    ipam:
      config:
        - subnet: 192.168.123.0/24        # Specify subnet
          gateway: 192.168.123.1          # Gateway address
          ip-range: 192.168.123.192/28    # Available IP addresses
```

This creates a new network named pihole_network using the parent network interface ovs_en0. This will be visible in the Synology docker UI under networks.

    Please check your network interface using ipconfig and change the parent device above to match.

Next we need to add our Pi-Hole container. This can be added with the following configuration.

```
services:
  pihole:
    container_name: pihole          # We name our container here
    image: pihole/pihole:latest     # Version 4.1 works with the required cap_add section
    hostname: pihole                # Containers hostname (optional)
    domainname: example.com         # Contaners domain (optional)
    mac_address: d0:ca:ab:cd:ef:01  # Random MAC address (optional)
    cap_add:
      - NET_ADMIN                   # This is required for version 4.1 onwards
    networks:
      - pihole_network              # Same name of network defined above
    dns:
      - 127.0.0.1
      - 8.8.8.8
    ports:
      - 443/tcp
      - 53/tcp
      - 53/udp
      - 67/udp
      - 80/tcp
    environment:                    # Optional environment configuration
      ServerIP: 192.168.123.199     # Change is this matches your NAS IP
      WEBPASSWORD: “”
      VIRTUAL_HOST: pihole.example.com
    restart: unless-stopped         # Set container to always restart
```

We specify our containers name, image and various networking information plus the environments required by Pi-Hole.

The combination of these two configurations are all that is required to create and run Pi-Hole on your Synology NAS. The complete file can be downloaded here.1

If you need more information the documentation can be found here.
How do we use docker compose

Assuming you have your docker compose file correctly setup (either writing your own or downloading one of mine) you can now start up Pi-Hole from the command line. If all works out then Pi-Hole should now be up and running and visible inside the Synology Docker UI.

```
$sudo docker-compose up -d
```

    Docker compose requires root access which will ask for your admin password. The -d is needed to run in daemon mode, if we do not supply this then the command will block in the shell.

How do we go about updating - you might ask?

Updating to the latest image is very easy. You can run the following commands.

```
$sudo docker-compose pull
$sudo docker-compose up -d
```

    Optionally you can specify the service $sudo docker-compose up pihole if you have mulitple services

This will download the new version and then re-create the updated Pi-Hole container.
Optional step; one file per container

As I have many containers running on my Synology I separated each service into it’s own file and included it along with it’s configuration and volumes. This way I can compose many docker files together rather than having one huge file. This is accomplished by the following in my main docker-compose.yaml file in the full example:

```
services:
  pihole:
    extends:
      service: pihole
      file: pihole/docker-compose.yaml
```

Downloading the combined2 file will give you the mulitple docker files, or you can download the single1 file for a basic simple container.

Head over to my project on gitlab to see all files in full 

---

## docker-compose extends

- [docker compose extends example](https://gitlab.com/tonyklawrence/files.tonylawrence.com/-/tree/master/synology/pihole/compose)

- docker-compose.yml

```
# Note: 192.168.123.xxx is an example network, you must update all these to match your own.

version: '2'

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    hostname: pihole
    domainname: example.com             # <-- Update
    mac_address: d0:ca:ab:cd:ef:01
    cap_add:
      - NET_ADMIN
    networks:
      pihole_network:
        ipv4_address: 192.168.123.199   # <-- Update
    dns:
      - 127.0.0.1
      - 8.8.8.8
    ports:
      - 443/tcp
      - 53/tcp
      - 53/udp
      - 67/udp
      - 80/tcp
    environment:
      ServerIP: 192.168.123.199         # <-- Update (match ipv4_address)
      VIRTUAL_HOST: pihole.example.com  # <-- Update (match hostname + domainname)
      WEBPASSWORD: ""                   # <-- Add password (if required)
    restart: unless-stopped

networks:
  pihole_network:
    driver: macvlan
    driver_opts:
      parent: ovs_eth0
    ipam:
      config:
        - subnet: 192.168.123.0/24            # <-- Update
          gateway: 192.168.123.1              # <-- Update
          ip_range: 192.168.123.192/28        # <-- Update

```
- combined/docker-compose.yml

```
version: '2'

services:
  pihole:
    extends:
      service: pihole
      file: dns/docker-compose.yaml
    networks:
      home:
        ipv4_address: 192.168.123.201

  another:
    extends:
      service: another
      file: another/docker-compose.yaml
    networks:
      home:
        ipv4_address: 192.168.123.202
        
networks:
  home:
    driver: macvlan
    driver_opts:
      parent: eth0
    ipam:
      config:
        - subnet: 192.168.123.0/24
          gateway: 192.168.123.1
          ip_range: 192.168.123/28
```

- combined/pihole/docker-compose.yml

```
version: '2'

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:4.0.0-1
    hostname: pihole
    domainname: my.network
    mac_address: d0:ca:ab:cd:ef:01
    cap_add:
      - NET_ADMIN
    ports:
      - 443/tcp
      - 53/tcp
      - 53/udp
      - 67/udp
      - 80/tcp
    environment:
      ServerIP: 1092.168.123.201
      WEBPASSWORD: ""
      VIRTUAL_HOST: pihole.my.network
      DNS1: 8.8.8.8
      DNS2: 8.8.4.4
    volumes:
      - /volume1/docker/dns/volume:/etc/pihole:rw
      - /volume1/docker/dns/config/hosts:/etc/hosts:ro
      - /volume1/docker/dns/config/resolv.conf:/etc/resolv.conf:ro
      - /volume1/docker/dns/config/dnsmasq.conf:/etc/dnsmasq.d/02-network.conf:ro
    restart: always
```


---

## Start / Stop Synology Package from the Command-Line

- [synology dsm 6/7](https://linuxhint.com/start-stop-synology-package-command-line/#a3)
- [references](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1269-768x80.png)

### Enabling SSH on Synology NAS:

To start and stop Synology packages from the command line, you will have to access the Terminal of your Synology NAS.
To access the Terminal of your Synology NAS, you will have to enable the SSH service of your Synology NAS.

To do that, open the Control Panel app and click on Terminal & SNMP as marked in the screenshot below.

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1242-768x500.png)

From the Terminal tab, check Enable SSH service and click on Apply.

![2](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1246-768x452.png)

Click on OK.

![3](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1248-768x450.png)

The SSH service of your Synology NAS should be enabled.

![4](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1250-768x453.png)

#### Connecting to the Synology NAS via SSH:

To access the Terminal of your Synology NAS, you will need to know the IP address of your Synology NAS.

You can find the IP address of your Synology NAS in the System Health widget as marked in the screenshot below. There are other methods of finding the IP address of your Synology NAS. For more information, read the article How Do I Find the IP Address of My Synology NAS.

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1256.png)

To access the Terminal of your Synology NAS via SSH, open a terminal program on your computer and run the following command:

```
$ ssh <username>@<ip-address>
```

Here, <username> is your Synology login username and <ip-address> is the DNS name or IP address of your Synology NAS.

In my case, the command is:

```
$ ssh shovon@192.168.0.120
```

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1257.png)

As you’re accessing the Terminal of your Synology NAS via SSH for the first time, you will be asked to verify the fingerprint of your Synology NAS. Type in yes and press <Enter> to verify the fingerprint.

![2](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1258-768x105.png)

Type in the login password of your Synology user and press <Enter>.

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1259-768x74.png)

You will be logged in to the Terminal of your Synology NAS via SSH. You can run any command you want here.

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1260-768x183.png)

#### Listing Installed Synology Packages:

You can list all the installed packages of your Synology NAS with the following command:

```
$ sudo synopkg list --name
```

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1261.png)

All the installed packages of your Synology NAS should be listed, as you can see in the screenshot below.

![2](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1262.png)

#### Checking the Status of Synology Packages:

You can check whether a Synology package Docker (let’s say) is stopped or running with the following command:

```
$ sudo synopkg status Docker
```

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1263.png)

As you can see, the package Docker is in started state. It means that the package is running.

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1264.png)

#### Stopping Synology Packages:

To stop a running Synology package Docker (let’s say), run the following command:

```
$ sudo synopkg stop Docker
```

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1265.png)

The Synology package Docker should be stopped.

You can also confirm that the Docker package is stopped by checking the package status with the following command:

```
$ sudo synopkg status Docker
```

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1267.png)

#### Starting Synology Packages:

To start a stopped Synology package Docker (let’s say), run the following command:

```
$ sudo synopkg start Docker
```

The Synology package Docker should be started.

![1](https://linuxhint.com/wp-content/uploads/2022/01/word-image-1269-768x80.png)

You can also confirm that the Docker package is running/started by checking the package status with the following command:

```
$ sudo synopkg status Docker
```

Conclusion:

I have shown you how to access the Terminal of your Synology NAS in this article. I have also shown you how to list the installed packages of your Synology NAS from the command line. I have also shown you how to start and stop Synology packages from the command line.

---

## SystemD ssh tunnel services

- [ssh tunnel as systemd service](https://ivanmorenoj.medium.com/ssh-tunnel-as-systemd-service-3c53bd157ac1)

SSH tunneling is a method of transporting arbitrary networking data over an encrypted SSH connection. It can be used to add encryption to legacy applications. It can also be used to implement VPNs (Virtual Private Networks) and access intranet services across firewalls.

SSH has 3 types of tunneling: local, remote and dynamic. Each of than can be use for different purpose.

### Local port forwarding

With local port forwarding we can forward remote port to local environment:

```
ssh -nNT -L 8000:remotehost:80 user@remotehost
```

In above example we forward remotehost:80 to local environment through 8000 port, then we can access to remotehost:80 typing localhost:8000 in our local environment

### Remote port forwarding

With remote port forwarding we can expose local environment to remote host

```
ssh -nNT -R 8080:localhost:80 user@remotehost
```

In above example we expose localhost:80 to remote environment through port 8080, assuming that remotehost is a server with public ip address, we can access to that service with [public_ip]:8080, but we need to enable some options in /etc/ssh/sshd_config

```
AllowTcpForwarding yes 
GatewayPorts yes
```

### Dynamic port forwarding

With dynamic port forwarding you can implement a system proxy

```
ssh -nNT -D 9090 user@remotehost
```

In above example we implement a system proxy via ssh tunnel, it can be configured in the system as socket proxy through 9090 port
SSH tunnel with systemd

SSH tunnels is an incredible feature, but what happens if we want to implement at startup of the system, we can do that with systemd service

First we need to setup a ssh keys on the remote server and create a template for each type of tunnel local, remote and dynamic and save in:

`/etc/systemd/system/ `

And environment variables, save in

`/etc/default/`

Example of dynamic, local and remote tunnel respectively

- /etc/systemd/system/ssh-tunnel@.service

```
[Unit]
Description=Setup a dynamic tunnel to %I
After=network.target

[Service]
EnvironmentFile=/etc/default/dynamic-tunnel@%i
ExecStart=/usr/bin/ssh -i ${PATH_TO_KEY} -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -nNT -D ${LOCAL_PORT} ${REMOTE_USER}@${REMOTE_HOST}
RestartSec=15
Restart=always
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

- /etc/default/dynamic-tunnel@stsproxy

```
PATH_TO_KEY=/home/db_user/.ssh/key-db-server
LOCAL_PORT=8080
REMOTE_ADDR=localhost
REMOTE_PORT=9090
REMOTE_USER=db_user
REMOTE_HOST=db-server
```

- local-tunnel@.service

```
[Unit]
Description=Setup a local tunnel to %I
After=network.target

[Service]
EnvironmentFile=/etc/default/local-tunnel@%i
ExecStart=/usr/bin/ssh -i ${PATH_TO_KEY} -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -nNT -L ${LOCAL_PORT}:${REMOTE_ADDR}:${REMOTE_PORT} ${REMOTE_USER}@${REMOTE_HOST}
RestartSec=15
Restart=always
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

- local-tunnel@database

```
PATH_TO_KEY=/home/db_user/.ssh/key-db-server
LOCAL_PORT=8080
REMOTE_ADDR=localhost
REMOTE_PORT=9090
REMOTE_USER=db_user
REMOTE_HOST=db-server
```

- remote-tunnel@.service

```
[Unit]
Description=Setup a remote tunnel to %I
After=network.target

[Service]
EnvironmentFile=/etc/default/remote-tunnel@%i
ExecStart=/usr/bin/ssh -i ${PATH_TO_KEY} -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -nNT -R ${REMOTE_PORT}:${LOCAL_ADDR}:${LOCAL_PORT} ${REMOTE_USER}@${REMOTE_HOST}
RestartSec=15
Restart=always
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

- remote-tunnel@testserver

```
PATH_TO_KEY=/home/gtw_user/.ssh/key-gtw-server
LOCAL_ADDR=localhost
LOCAL_PORT=80
REMOTE_PORT=80
REMOTE_USER=gtw-user
REMOTE_HOST=gtw-server
```

And finally for activate this services, type the next command for dynamic, local and remote tunnel respectively

```
systemctl enable --now dynamic-tunnel@sysproxy
systemctl enable --now local-tunnel@database
systemctl enable --now remote-tunnel@testserver
```
---

## GVFS Gnome Virtual FileSystem

- [mount remote servers via gvfs sshfs](https://linux-commander.com/using-gvfs-to-access-remote-servers-via-ftpsftp/)

Using the Gnome virtual file system (gvfs) packages allows us to access remote servers from the linux userspace GUI environment via FTP/Obex/SSH/WebDAV/WebDAVS/Samba

The gigolo package allows management of the gvfs mounts using a GUI application.  If not installed with your distro, you can install it from the base repositorities using either :

```
# sudo apt-get install gigolo     for debian/ubuntu
```

or
```
# yum -y install gigolo               for red hat/centos/fedora
```
Starting gigolo – Look for the launcher for gigolo under the System (debian) /System Tools (red hat/centos) menu category.  To get started click Connect.[1]

![screenshot_00030](https://linux-commander.com/wp-content/uploads/2016/11/screenshot_00030-150x150.png)

Choose which protocol you want to use to mount your remote file system [1]. Supported options are FTP/Obex/SSH/WebDAV/WebDAVS/Samba.  For my system, I want to mount my remote Digital Ocean web server, so I select SSH [2], and then enter my SSH username and port [3]. Once I click Connect, my new gvfs bookmark appears in the right pane [4] and double clicking on it opens up the mounted file system in nautilus [5].

![1](https://linux-commander.com/wp-content/uploads/2016/11/screenshot_00026-300x293.png)

![2](https://linux-commander.com/wp-content/uploads/2016/11/screenshot_00025-300x293.png)

![3](https://linux-commander.com/wp-content/uploads/2016/11/screenshot_00027-300x293.jpg)

![4](https://linux-commander.com/wp-content/uploads/2016/11/screenshot_00029-2-300x261.png)

![5](https://linux-commander.com/wp-content/uploads/2016/11/screenshot_00028-1-232x300.png)

Great! Now that the file systems are mounted up, you can use the GUI tools to navigate the remote file systems, modify files, etc.  But what if you’re a command line junkie?debian: from the shell,I can cd to $XDG_RUNTIME_DIR/gvfs to get to my mounted remote file systems.

```
glaw@fedora:1000$ cd $XDG_RUNTIME_DIR/gvfs
glaw@fedora:gvfs$ ls
sftp:host=web2.XXXXXX,port=XXX,user=root
glaw@fedora:gvfs$ cd sftp\:host\=web2.XXXXX\,port\=XXXX\,user\=root/
glaw@fedora:sftp:host=web2.XXXXX,port=XXXX,user=root$ ls -lart
total 524
drwx------ 1 glaw glaw  16384 Jul  8  2014 lost+found
-rw-r--r-- 1 glaw glaw      0 Jul  8  2014 .autorelabel
drwxr-xr-x 1 glaw glaw   4096 Aug 12  2015 srv
drwxr-xr-x 1 glaw glaw   4096 Aug 12  2015 mnt
drwxr-xr-x 1 glaw glaw   4096 Aug 12  2015 media
drwxr-xr-x 1 glaw glaw   4096 Feb 29  2016 usr
drwxr-xr-x 1 glaw glaw   4096 May 16 11:25 opt
dr-xr-xr-x 1 glaw glaw   4096 Jul  5 08:54 lib
drwxr-xr-x 1 glaw glaw   4096 Aug  1 03:38 backup
drwxr-xr-x 1 glaw glaw   4096 Sep 24 23:14 home
dr-xr-xr-x 1 glaw glaw      0 Sep 25 01:11 proc
dr-xr-xr-x 1 glaw glaw      0 Sep 25 01:11 sys
drwxr-xr-x 1 glaw glaw   4096 Sep 25 01:11 var
drwxr-xr-x 1 glaw glaw   2880 Sep 25 01:11 dev
dr-xr-xr-x 1 glaw glaw   4096 Sep 25 01:11 .
dr-xr-xr-x 1 glaw glaw   4096 Sep 25 01:11 boot
drwxr-xr-x 1 glaw glaw   4096 Oct 24 13:35 etc
dr-x------ 3 glaw glaw      0 Oct 25 13:15 ..
dr-xr-xr-x 1 glaw glaw  36864 Oct 26 13:56 lib64
dr-xr-xr-x 1 glaw glaw  36864 Oct 26 13:56 bin
dr-xr-x--- 1 glaw glaw   4096 Oct 26 14:00 root
dr-xr-xr-x 1 glaw glaw  12288 Nov  1 08:41 sbin
drwxr-xr-x 1 glaw glaw   4096 Nov  6 20:53 scripts
drwxr-xr-x 1 glaw glaw    680 Nov  7 00:00 run
drwxrwxrwx 1 glaw glaw 372736 Nov  7 14:04 tmp
```

Connecting from the command line
The Linux GUI is nice but it is also to mount this up via the command line.

```
debian:  # sudo apt-get install sshfs
red hat: # yum install sshfs
# sshfs -p 1234  root@web2.XXXXX.com:/ /path/to/mount
To enable this at login, you can also add the following to your GUI user's crontab : 
# crontab -e
@reboot sshfs -p 1234  root@web2.XXXXX.com:/ /path/to/mount
```

Ok, that’s great but what are the uses?

As a web developer occasionally I have to try to track down where a certain CSS class definition is or where a PHP function is.  When I haev SSH access, this is no problem, but with FTP only, it makes it much more difficult unless I want to mirror a full website on my local computer.  With a gvfs FTP mount, it makes it easy to cd over to the mount point and then use the handy dandy grep command to find what I am looking for :

```
# grep -r myCSSClass ./
```

---

## SystemD ssh tunnel

- [systemd ssh tunnel](https://gist.github.com/drmalex07/c0f9304deea566842490)

```
Create a template service file at /etc/systemd/system/secure-tunnel@.service. The template parameter will correspond to the name of target host:

[Unit]
Description=Setup a secure tunnel to %I
After=network.target

[Service]
Environment="LOCAL_ADDR=localhost"
EnvironmentFile=/etc/default/secure-tunnel@%i
ExecStart=/usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L ${LOCAL_ADDR}:${LOCAL_PORT}:localhost:${REMOTE_PORT} ${TARGET}

# Restart every >2 seconds to avoid StartLimitInterval failure
RestartSec=5
Restart=always

[Install]
WantedBy=multi-user.target

We need a configuration file (inside /etc/default) for each target host we will be creating tunnels for. For example, let's assume we want to tunnel to a host named jupiter (probably aliased in /etc/hosts). Create the file at /etc/default/secure-tunnel@jupiter:

TARGET=jupiter
LOCAL_ADDR=0.0.0.0
LOCAL_PORT=20022
REMOTE_PORT=22

Note that for the above to work we need to have allready setup a password-less SSH login to target (e.g. by giving access to a non-protected private key).

Now we can start the service instance:

systemctl start secure-tunnel@jupiter.service
systemctl status secure-tunnel@jupiter.service

Or enable it, so it get's started at boot time:

systemctl enable secure-tunnel@jupiter.service
```

- [systemd ssh tunnel](https://github.com/renxida/labtunnel)

```
SSH Tunnel For SystemD

This systemd unit maintains an SSH tunnel of your choice. No root required.
Install

A systemd unit template file needs to be installed in order to run tunnels.

Running install.sh copies labtunnel@.service to your ~/.config/systemd/user . Do not use sudo.

Read man systemd.unit for more information on templates.
Defining Tunnels

To define a tunnel, add the following lines to your ~/.ssh/config:

Host [Your tunnel name]
    HostName [ip or url to remote computer]
    Port     [ssh port, usually 22]
    User     xren
    IdentityFile ~/.ssh/zlab.key
    LocalForward [local port to access your tunnel] localhost:[remote port you want to connect to]

Read man ssh_config for more info.
Running Tunnels

To start your tunnel, substitute the tunnel name you defined into the following command: systemctl --user start labtunnel@[your tunnel name]

To set-up autostart, use the same command but replace start with enable: systemctl --user start labtunnel@[your tunnel name]

See more at man systemctl
Uninstalling

Run uninstall.sh from the project directory. The script disable/removes autostart tunnels and the installed systemd template file.

After uninstalling, if you don't delete the lines you added to your ssh config, you may still run ssh [your tunnel name] to access the tunnels manually.
Source

This project is based off drmalex07's systemd ssh tunneler at https://gist.github.com/drmalex07/c0f9304deea566842490.

My improvements include:

    No priviledges required: everything is installed in ~/.config/systemd/user (your user's personal systemd-related files)
    Install/uninstall scripts
    Usses ssh_config: can define all sorts of ssh connections, not just local tunnels
```

- [systemd ssh jumphost + autossh tunnel](https://gist.github.com/linuxmalaysia/05228a52feea1b117236a8a0d2373ade)

```
1) ==== Autossh using systemd ====

Example from

https://gist.github.com/drmalex07/c0f9304deea566842490

2) =============

Install autossh

3) ==== /etc/default/secure-tunnel@yourjumpsshserver == change yourjumpsshserver to your ip or dnsname

TARGET=yourjumpsshserver
LOCAL_ADDR=0.0.0.0
LOCAL_PORT=22
# port that will be use to ssh at remote server
REMOTE_PORT=54322
# change user as per remote server
USERNAME=user
# change SSH port used at jump server
SSH_TARGET_PORT=22


4) ==== /etc/systemd/system/secure-tunnel@.service

[Unit]
Description=Setup a secure tunnel to %I
After=network.target

[Service]
Environment="LOCAL_ADDR=localhost"
EnvironmentFile=/etc/default/secure-tunnel@%i
###ExecStart=/usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L ${LOCAL_ADDR}:${LOCAL_PORT}:localhost:${REMOTE_PORT} ${USERNAME}@${TARGET}
Environment="AUTOSSH_GATETIME=0"
ExecStart=/usr/bin/autossh -M 0 -o "ExitOnForwardFailure=yes" -o "ServerAliveInterval 30" -o "ServerAliveCountMax 3" -NR ${REMOTE_PORT}:${LOCAL_ADDR}:${LOCAL_PORT} -p ${SSH_TARGET_PORT} ${USERNAME}@${TARGET}

# Restart every >2 seconds to avoid StartLimitInterval failure
RestartSec=5
Restart=always

[Install]
WantedBy=multi-user.target

6) ===== SSH key copy to yourjumpsshserver

As root (systemd running as root)

ssh-copy-id user@yourjumpsshserver -p 443

You may need to run ssh-keygen. Make sure ls /root/.ssh is empty

5) ============ systemctl

systemctl daemon-reload
systemctl status secure-tunnel@yourjumpsshserver
systemctl enable secure-tunnel@yourjumpsshserver
systemctl start secure-tunnel@yourjumpsshserver
systemctl stop secure-tunnel@yourjumpsshserver

==== Autossh using systemd ====
```

-[systemd ssh tunnels](https://gist.github.com/sunnyszy/7b62272a478fcda8cafd2e97a5147ebb)

```
README

Create a template service file at /etc/systemd/system/secure-tunnel@.service. The template parameter will correspond to the name of target host:

[Unit]
Description=Setup a secure tunnel to %I
After=network.target

[Service]
Environment="LOCAL_ADDR=localhost"
EnvironmentFile=/etc/default/secure-tunnel@%i
ExecStart=/usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L ${LOCAL_ADDR}:${LOCAL_PORT}:localhost:${REMOTE_PORT} ${TARGET}
User=zhenyus
Group=zhenyus

# Restart every >2 seconds to avoid StartLimitInterval failure
RestartSec=5
Restart=always

[Install]
WantedBy=multi-user.target

We need a configuration file (inside /etc/default) for each target host we will be creating tunnels for. For example, let's assume we want to tunnel to a host named jupiter (probably aliased in /etc/hosts). Create the file at /etc/default/secure-tunnel@jupiter:

TARGET=jupiter
LOCAL_ADDR=0.0.0.0
LOCAL_PORT=20022
REMOTE_PORT=22

Note that for the above to work we need to have allready setup a password-less SSH login to target (e.g. by giving access to a non-protected private key).

Now we can start the service instance:

systemctl start secure-tunnel@jupiter.service
systemctl status secure-tunnel@jupiter.service

Or enable it, so it get's started at boot time:

systemctl enable secure-tunnel@jupiter.service
```

- [systemd home assistent ssh tunnel](https://github.com/AlexxIT/hassio-addons/tree/master/ssh_tunnel)

---

### SSH Tunnel

Hass.io addon for configuring external access to **Home Assistant** via an external server via SSH protocol. Useful if you have a "gray" IP address.

To work, you must already have a rented server with SSH access.

By default, the addon forwards ports 80 and 443 of the Hass.io server to an external server. It also creates a SOCKS proxy on the Hass.io server.

The first time you set it up, just configure the `host`, `port` and `user` of your server.

When run, the public SSH key (rather long) will be displayed in the logs, for example:

```
ssh-rsa AAAAB3NzaC1yc2EA...XfsAODObXZRVMI03 root@2fc7ce79c3de
```

The key is randomly generated every time the addon is installed. It must be **copied to your public server** in the home directory of the user on whose behalf you are connecting to the server via SSH:

`~/.ssh/authorized_keys`

For example `/root/.ssh/authorized_keys` or `/home/USERNAME/.ssh/authorized_keys`

If necessary, create a `.ssh` directory and an `authorized_keys` file - they may not be there.

**Warning!** The `.ssh` directory is considered hidden.

After a break - the connection will be restored automatically. But read the section **Discontinuity Recovery**.

#### Settings

- `ssh_host` - public server address
- `ssh_user` - public server user
- `ssh_port` - public server port
- `tunnel_http` - redirect HTTP port
- `tunnel_https` - redirect HTTPS port
- `socks_port` - enable SOCKS proxy mode on the specified port of the Hass.io server
- `advanced` - additional ssh command options
- `before` - see section **Crash Recovery**

**Attention!** To forward "privileged" ports (for example, 80 and 443), the public server user must be an administrator. If you have problems with this, you can google: **ssh forward privileged ports**.

Config example:

- forwards ports 80 and 443 from the Hass.io server to an external server
- creates a SOCKS proxy on the Hass.io server on port 1080

```yaml
ssh_host: 87.250.250.242
ssh_user:root
ssh_port: 22
tunnel_http: true
tunnel_https: true
socks_port: 1080
```

#### Application

##### External access to Home Assistant

It is convenient to use in conjunction with [Caddy Proxy](https://github.com/bestlibre/hassio-addons/tree/master/caddy_proxy) addon for hass.io. This is a lightweight web server that automatically generates HTTPS certificates.

Caddy Proxy config example:

```yaml
homeassistant: sshtunnel.duckdns.org
vhosts": []
raw_config": []
email": sshtunnel@gmail.com
```

E-mail is used when generating certificates. The domain name (DNS) must be directed to the IP address of your public server.

**Setting:**

1. Rent a VDS server
2. Set up the SSH Tunnel addon
3. Create and set up a Duck DNS account
4. Set up the Caddy Proxy addon
5. Set up two-factor authentication for Home Assistant
6. Enjoy secure and stable external access to HA

##### Proxy for Telegram bot

```yaml
telegram_bot:
-platform:broadcast
  api_key: TELEGRAM_BOT_API_KEY
  allowed_chat_ids: [123456789]
  proxy_url: socks5://172.17.0.1:1080
```

PS: `172.17.0.1` - you don't need to change this IP for a standard hass.io installation

##### Tunnel for any local resources

For example, external access to a home OpenVPN server running on a router.

```yaml
advanced: -R 1194:192.168.1.1:1194
```

PS: Now you can connect to your home at `sshtunnel.duckdns.org:1194`

##### Breakdown recovery

The addon is configured to check the connection every 30 seconds. With 3 unsuccessful checks, the connection is terminated.

**Attention!** With the default settings, after the connection is broken, the ports on your server will remain busy!

Option two:

1. Configure client connection check on the server: [ClientAliveInterval](https://sys-adm.in/os/nix/429-centos-increase-ssh-session-timeout.html)

2. Release the port at the beginning of each client connection

To do this, I made the option `before: fuser -k 443/tcp`. **fuser** is one way to free up a busy port in Ubuntu, **not installed by default!**

Personally, I use the second option.

---

## SystemD / SystemCtl automount/mount SSHFS/NFS

- [tomecek systemd automoun nfs](https://blog.tomecek.net/post/automount-with-systemd/)

You need two files

  - First one to setup the mount itself.
  - Second to perform automatic mounting.


```
$ cat /etc/systemd/system/mnt-scratch.automount
[Unit]
Description=Automount Scratch

[Automount]
Where=/mnt/scratch

[Install]
WantedBy=multi-user.target
```

```
$ cat /etc/systemd/system/mnt-scratch.mount
[Unit]
Description=Scratch

[Mount]
What=nfs.example.com:/export/scratch
Where=/mnt/scratch
Type=nfs

[Install]
WantedBy=multi-user.target
```

```
# systemctl daemon-reload

# systemctl is-enabled mnt-scratch.mount
disabled
# systemctl is-enabled mnt-scratch.automount                                                                                             1 ↵
enabled
# systemctl start mnt-scratch.automount                                                                                             1 ↵
# ls /mnt/scratch >/dev/null
# systemctl status mnt-scratch.automount
● mnt-scratch.automount - Automount Scratch
   Loaded: loaded (/etc/systemd/system/mnt-scratch.automount; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2016-04-18 10:49:04 CEST; 4h 33min ago
    Where: /mnt/scratch

Apr 18 10:49:04 oat systemd[1]: Set up automount Automount Scratch.
Apr 18 10:49:14 oat systemd[1]: mnt-scratch.automount: Got automount request for /mnt/scratch, triggered by 20266 (zsh)
# systemctl status mnt-scratch.mount
● mnt-scratch.mount - Scratch
   Loaded: loaded (/proc/self/mountinfo; disabled; vendor preset: disabled)
   Active: active (mounted) since Mon 2016-04-18 10:49:16 CEST; 4h 33min ago
    Where: /mnt/scratch
     What: nfs.example.com:/export/scratch

Apr 18 10:49:14 oat systemd[1]: Mounting Scratch...
Apr 18 10:49:16 oat systemd[1]: Mounted Scratch.
```

- [gist systemd automount sshfs](https://gist.github.com/proprietary/96f6f08758fb98da8467880904191f64)

```
# Change the relevant {{ PARTS OF THIS FILE }} for your remote address etc.
# Make sure this unit file is named similarly to your mountpoint; e.g., for /mnt/mymountpoint name this file mnt-mymountpoint.mount
# On Ubuntu:
# $ sudo cp mnt-mymountpoint.mount /lib/systemd/system/
# $ sudo systemctl enable mnt-mymountpoint.mount
# $ sudo systemctl start mnt-mymountpoint.mount
# On Fedora:
# $ sudo cp mnt-mymountpoint.mount /etc/systemd/system
# $ sudo systemctl enable mnt-mymountpoint.mount
# $ sudo systemctl start mnt-mymountpoint.mount

[Unit]
Description=Mount my remote filesystem over sshfs with fuse

[Install]
WantedBy=multi-user.target

[Mount]
What={{ USER }}@{{ HOST }}:{{ REMOTE DIR }}
Where={{ MOUNTPOINT like /mnt/mymountdir }}
Type=fuse.sshfs
# I recommend using your SSH key (no password authentication) with the following options so that you don't have to mount every time you boot
Options=_netdev,allow_other,IdentityFile=/home/{{ MY LOCAL USER WITH SSH KEY IN ITS HOME DIRECTORY }}/.ssh/id_rsa,reconnect,x-systemd.automount,uid=1000,gid=1000
# Change to your uid and gid as well according to the output of `cat /etc/group`
```


## Install Rancher on Docker Single Node

date: 2022-04-19
by: alpha

- [https://rancher.com/docs/rancher/v2.5/en/installation/other-installation-methods/single-node-docker/](https://rancher.com/docs/rancher/v2.5/en/installation/other-installation-methods/single-node-docker/)

```bash
# opsi a: default rancher-generated self-signed certificate
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  rancher/rancher:latest

# opsi b: menggunakan self-signed certificate yg sudah ada
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
  -v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
  -v /<CERT_DIRECTORY>/<CA_CERTS.pem>:/etc/rancher/ssl/cacerts.pem \
  --privileged \
  rancher/rancher:latest

# opsi c: menggunakan cert yg sudah di signed online (berbayar)
# opsi --no-cacerts mencegah rancher generate ca baru
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  -v /<CERT_DIRECTORY>/<FULL_CHAIN.pem>:/etc/rancher/ssl/cert.pem \
  -v /<CERT_DIRECTORY>/<PRIVATE_KEY.pem>:/etc/rancher/ssl/key.pem \
  --privileged \
  rancher/rancher:latest \
  --no-cacerts

# opsi d: letsencrypt, otomatis, perlu domain aktif
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  rancher/rancher:latest \
  --acme-domain <YOUR.DNS.NAME>
```