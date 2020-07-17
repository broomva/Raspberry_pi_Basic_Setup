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
