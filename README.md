# Raspberry_pi_Basic_Setup

## Change default password and others
```
raspi-config
```

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

## Define Personal Services Exec
```
sudo nano /lib/systemd/system/sample.service

 [Unit]
 Description=My Sample Service
 After=multi-user.target

 [Service]
 Type=idle
 ExecStart=/usr/bin/python /home/pi/sample.py

 [Install]
 WantedBy=multi-user.target
 
sudo chmod 644 /lib/systemd/system/sample.service
```
configure systemd:
```
sudo systemctl daemon-reload
sudo systemctl enable sample.service
sudo reboot -n



## Install Requirements
From previous iteration, stpf transfered:
```
pip install requirements.txt
```

## Install Requirements
From previous iteration, stpf transfered:
```
pip install requirements.txt
```

## Install Mosquitto MQTT server

```
sudo apt install -y mosquitto mosquitto-clients
sudo systemctl enable mosquitto.service

#sudo systemctl restart mosquitto.service
```
and add password:
```
sudo systemctl stop mosquitto.service
sudo mosquitto_passwd -c /etc/mosquitto/passwd <user_name>
# mosquitto_passwd -b passwordfile user password #to change password

sudo nano /etc/mosquitto/mosquitto.conf
# and add
password_file /etc/mosquitto/passwd
allow_anonymous false

mosquitto -c /etc/mosquitto/mosquitto.conf
#sudo systemctl start mosquitto.service
```

check on terminal with
```
mosquitto_sub -h host -p 1883 -t '#' -u user -P password
# connects to all topics
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

