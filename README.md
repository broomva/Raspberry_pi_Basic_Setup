# Raspberry_pi_Basic_Setup

## Setup Static IP


``` bash
sudo service dhcpcd status
sudo service dhcpcd start
sudo systemctl enable dhcpcd
sudo nano /etc/dhcpcd.conf

interface eth0
static ip_address=192.168.0.4/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1

sudo reboot
```

## Install Requirements
From previous iteration, stpf transfered:
```
pip install requirements.txt
```



## Installing and Upgrading Node-RED
```
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
```
This script will:

remove the pre-packaged version of Node-RED and Node.js if they are present
install the current Node.js LTS release using the NodeSource. If it detects Node.js is already installed from NodeSource, it will ensure it is at least Node 8, but otherwise leave it alone
install the latest version of Node-RED using npm
optionally install a collection of useful Pi-specific nodes
setup Node-RED to run as a service and provide a set of commands to work with the service

```
node-red-pi --max-old-space-size=256
sudo systemctl enable nodered.service
# sudo systemctl disable nodered.service
```
Port 1880 default

## Install Docker
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh 
```

