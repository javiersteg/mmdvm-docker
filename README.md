CREDTIS TO -->>
 MMDVM by G4KLX -->>

I have modified it to be used on debian bookworm.
I adapted it to use last mmdvm version.

Can run on every GNU/linux machine with docker.
That means that include Orange Pi, and other ARM archs. Just need to install docker. 
For installation best is find the way that fits better with your OS... (Debian, Red Hat, Arch....) Yes. I'm saying that this can be installed for example on your OpenWRT router. But requires to be powerfull... For operating the DMR link and for build all the images.

If you do not have idea how to do this. Ask me over telegram @javiersteg.
Sure i will prepare an image for your architecture if it's more less common.

IMPORTANT= For this test i've used a Baofeng DM-1701 with OpenGD77 (2025/02/07 Beta Firmware).
So my connection is detected as a Serial UART TTY. (/dev/ttyACM0)
You should need to change to the proper one that fits your device.

Remember change common values for your needs. Callsign, ID, DMR or YSF master server.....

I have fixed and error with the path on Dockerfile.mmdvmdashboard that was giving an error when doing CP.

Steps for get it working are:
1-Get DOCKER WORKING and COMPOSE
2- FOR Run MMDVMHost, YSFGateway and MMDVM-Dashboard
```console
$ git clone https://github.com/f4hlv/mmdvm-docker.git
$ cd mmdvm-docker
```
Edit docker-compose.yml and run
Edit your radio and DMR/YSF parameters to fit your needs, TX/RX frecuencies and other parameters...
These files are inside config folder inside the path you are now.

```console
$ docker network create webgateway
$ docker-compose up -d
```

If you enter to the MMDVM file, there are some trace=0 and debug=0. So, if you are getting errors, good way to solve it is increase verbosity and launch docker without the "-d" argument, to avoid run it on background and be able to see realtime logs.

Good Luck.

## Port
- `80:80` Port configuration. "Port host":80(container port)
## Remove setup.php
```console
$ docker exec mmdvm-dashboard rm /var/www/html/setup.php
```
# Update
```console
$ docker-compose build --no-cache
$ docker-compose up -d
```

Thanks for all these progress G4KLX


Less important things.

I'm doing this FORK because i have a Orange Pi 3 Zero.
And i have seen that WPSD is not compatible with Orange PI, just raspberry.
So i decided create this fork and fix this problem.

My recomendation it's do this on a secure network with good walls on your network. IDK if this DMR gateway is secure and i cannot guarantee.


![logo](https://www.rs-online.com/designspark/rel-assets/dsauto/temp/uploaded/MMDVM.jpg?w=815)

# Install Docker
```console
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

# Install docker-compose
* (Debian Buster)
```console
$ sudo apt install docker-compose
```
* (Debian stretch)
```console
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
```
* (Raspberry Buster)
```console
$ sudo apt install docker-compose
```
* (Raspberry stretch)
```console
$ sudo apt-get -y install python-setuptools
$ sudo easy_install pip && sudo pip install docker-compose
```

# Run MMDVMHost, YSFGateway and MMDVM-Dashboard
```console
$ git clone https://github.com/f4hlv/mmdvm-docker.git
$ cd mmdvm-docker
```
Edit docker-compose.yml and run
```console
$ docker network create webgateway
$ docker-compose up -d
```
## Volume
- `/MMDVMHost/MMDVM.ini:ro` Path to the MMDVM.ini File
- `/MMDVMHost/RSSI.dat:ro` Path to the RSSI.dat File (Optional)
- `/MMDVMHost/DMRIds.dat:ro` Path to the DMRIds.dat File (Optional)
- `/YSFClients/YSFGateway/YSFHosts.txt:ro` Path to the YSFHosts.txt File (Optional)

example for MMDVM.ini(raspberry): /home/pi/MMDVM.ini:/MMDVMHost/MMDVM.ini:ro

# Dashboard
Configure options and destination directories `/etc/mmdvm` and `/etc/YSFGateway`. Then delete the setup.php file
## Port
- `80:80` Port configuration. "Port host":80(container port)
## Remove setup.php
```console
$ docker exec mmdvm-dashboard rm /var/www/html/setup.php
```
# Update
```console
$ docker-compose build --no-cache
$ docker-compose up -d
```

# docker-compose
```yml
version: '3.2'
services:
#############################################################################################
  mmdvmhost:
    build:
      context: .
      dockerfile: Dockerfile.mmdvmhost
    container_name: mmdvmhost
    volumes:
      - ./config/MMDVM.ini:/MMDVMHost/MMDVM.ini:ro
#      - ./config/RSSI.dat:/MMDVMHost/RSSI.dat:ro
#      - ./config/DMRIds.dat:/MMDVMHost/DMRIds.dat:ro
      - mmdvmhost:/MMDVMHost
      - log-mmdvm:/var/log/mmdvm
    networks:
      mmdvm:
        ipv4_address: 10.10.1.2      
    devices:
      - /dev/ttyACM0:/dev/ttyACM0
    restart: unless-stopped
#############################################################################################      
  ysfgateway:
    build:
      context: .
      dockerfile: Dockerfile.ysfgateway
    container_name: ysfgateway
    restart: unless-stopped
    volumes:
      - ./config/YSFGateway.ini:/YSFClients/YSFGateway/YSFGateway.ini:ro
#      - ./config/YSFHosts.txt:/YSFClients/YSFGateway/YSFHosts.txt:ro
      - log-YSFGateway:/var/log/YSFGateway
    depends_on:
      - mmdvmhost
    networks:
      mmdvm:
        ipv4_address: 10.10.1.3
#############################################################################################        
  ysf2dmr:
    build:
      context: .
      dockerfile: Dockerfile.ysf2dmr
    container_name: ysf2dmr
    volumes:
      - ./config/YSF2DMR.ini:/MMDVM_CM/YSF2DMR/YSF2DMR.ini:ro
#      - ./config/TGList-DMR.txt:/MMDVM_CM/YSF2DMR/TGList-DMR.txt:ro
    depends_on:
      - ysfgateway
    networks:
      mmdvm:
        ipv4_address: 10.10.1.30
    restart: unless-stopped        
#############################################################################################
  mmdvm-dashboard:
    build:
      context: .
      dockerfile: Dockerfile.mmdvmdashboard
    container_name: mmdvm-dashboard
    restart: always
    ports:
      - "80:80"
    volumes:
      - mmdvmhost:/MMDVMHost:ro
      - ./config:/config:ro
      - ./config/config.php:/var/www/html/config/config.php
      - log-mmdvm:/var/log/mmdvm:ro
      - log-YSFGateway:/var/log/YSFGateway:ro
    pid: host
    restart: always
    networks:
      - webgateway
#############################################################################################
volumes:
  mmdvmhost:
  log-mmdvm:
  log-YSFGateway:
  
networks:
  webgateway:
    external:
      name: webgateway
  mmdvm:
    ipam:
      driver: default
      config:
        - subnet: 10.10.1.0/24
```

This software is licenced under the GPL v2 and is intended for amateur and educational use only. Use of this software for commercial purposes is strictly forbidden.
