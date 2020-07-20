# Raspberry_pi_Basic_Setup

## Enable SSH beforehand
Create an empty file called ssh in the boot directory. This enables SSH so that you can log in remotely.

The next step is only required if you want the Raspberry Pi to connect to your wireless network. Otherwise, connect the it to your network by using a network cable.

(Optional) Create a file called wpa_supplicant.conf in the boot directory:

ctrl_interface=/var/run/wpa_supplicant
update_config=1
country=<Insert 2 letter ISO 3166-1 country code here>

network={
 ssid="<Name of your WiFi>"
 psk="<Password for your WiFi>"
}
All the necessary files are now on the SD card. Let’s start up the Raspberry Pi.

Eject the SD card and insert it into the SD card slot on the Raspberry Pi.
Connect the power cable and make sure the LED lights are on.
Find the IP address of the Raspberry Pi. Usually you can find the address in the control panel for your WiFi router.
Connect remotely via SSH
Open up your terminal and enter the following command:
ssh pi@<ip address>
SSH warns you that the authenticity of the host can’t be established. Type “yes” to continue connecting.
When asked for a password, enter the default password: raspberry.
Once you’re logged in, change the default password:
passwd

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

## Autostart scripts
Use Crontab Pref
```
sudo nano /home/pi/.bashrc
sudo nano /etc/rc.local
sudo nano /etc/profile
crontab -e
 @reboot python3 /home/pi/script.py
 @reboot sh /home/pi/in.sh
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

```

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


## Install Thingsboard
```
sudo apt update
sudo apt install openjdk-8-jdk
sudo update-alternatives --config java
java -version
# openjdk version "1.8.0.xxx

wget https://github.com/thingsboard/thingsboard/releases/download/v3.0.1/thingsboard-3.0.1.deb
sudo dpkg -i thingsboard-3.0.1.deb
```


## Configure PostgreSQL
```
# install **wget** if not already installed:
sudo apt install -y wget

# import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# add repository contents to your system:
RELEASE=$(lsb_release -cs)
echo "deb http://apt.postgresql.org/pub/repos/apt/ ${RELEASE}"-pgdg main | sudo tee  /etc/apt/sources.list.d/pgdg.list

# install and launch the postgresql service:
sudo apt update
sudo apt -y install postgresql-12
sudo service postgresql start

# Or
sudo apt install postgresql libpq-dev postgresql-client postgresql-client-common -y

sudo su - postgres
psql
\password
\q

# Ctrl+D” to return to main user console and connect to the database to create thingsboard DB:

psql -U postgres -d postgres -h 127.0.0.1 -W
CREATE DATABASE thingsboard;
\q


sudo nano /etc/thingsboard/conf/thingsboard.conf

# DB Configuration 
export DATABASE_ENTITIES_TYPE=sql
export DATABASE_TS_TYPE=sql
export SPRING_JPA_DATABASE_PLATFORM=org.hibernate.dialect.PostgreSQLDialect
export SPRING_DRIVER_CLASS_NAME=org.postgresql.Driver
export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=PUT_YOUR_POSTGRESQL_PASSWORD_HERE
export SPRING_DATASOURCE_MAXIMUM_POOL_SIZE=5
# Specify partitioning size for timestamp key-value storage. Allowed values: DAYS, MONTHS, YEARS, INDEFINITE.
export SQL_POSTGRES_TS_KV_PARTITIONING=MONTHS

# Update ThingsBoard memory usage and restrict it to 256MB in /etc/thingsboard/conf/thingsboard.conf
export JAVA_OPTS="$JAVA_OPTS -Xms256M -Xmx256M"



sudo nano /etc/thingsboard/conf/thingsboard.yml
Comment ‘# HSQLDB DAO Configuration’ block.
Uncomment ‘# PostgreSQL DAO Configuration’ block. Be sure to update the postgres databases username and password in the bottom two lines of the block (here, as shown, they are both “postgres”).

Change MQTT port to 1882

```
After configuring postgresql to thingsboard, install and load demo:
```
# --loadDemo option will load demo data: users, devices, assets, rules, widgets.
sudo /usr/share/thingsboard/bin/install/install.sh --loadDemo

sudo service thingsboard start

sudo systemctl enable thingsboard
sudo systemctl status thingsboard

/var/log/thingsboard
cat /var/log/thingsboard/thingsboard.log | grep ERROR

```


## Configure PostgreSQL to allow remote connection
```
# By default PostgreSQL is configured to be bound to “localhost”

netstat -nlt
# port 5432 is bound to 127.0.0.1. It means any attempt to connect to the postgresql server from outside the machine will be refused.

find / -name "postgresql.conf"

# Open postgresql.conf file and replace line

listen_addresses = 'localhost'
# with

listen_addresses = '*' # Remove #

# now configure the hba,

find / -name "pg_hba.conf"
open pg_hba.conf and add following entry at the very end.

host    all             all              0.0.0.0/0                       md5
host    all             all              ::/0                            md5


sudo reboot -n
netstat -nlt # now service on port 5432 shows open to all 0.0.0.0
```

## Intall Nginx 

```
sudo apt-get install nginx
sudo rm /etc/nginx/sites-enabled/default
sudo nano /etc/nginx/sites-available/node

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_set_header   X-Forwarded-For $remote_addr;
        proxy_set_header   Host $http_host;
        proxy_pass         "http://127.0.0.1:1337";
    }
}

sudo ln -s /etc/nginx/sites-available/node /etc/nginx/sites-enabled/node
sudo service nginx restart
```
example:
```
sudo nano /etc/nginx/sites-available/node

  GNU nano 2.9.3           /etc/nginx/sites-available/node
  server {
    server_name digitalinsa.com www.digitalinsa.com;

    location / {
        proxy_set_header   X-Forwarded-For $remote_addr;
        proxy_set_header   Host $http_host;
        proxy_pass         "http://127.0.0.1:8080";

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }

    location /grafana {
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header Host $http_host;
        proxy_pass      "http://127.0.0.1:3000";
        }
        
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/digitalinsa.com/fullchain.pem; # mana$
    ssl_certificate_key /etc/letsencrypt/live/digitalinsa.com/privkey.pem; # ma$
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    add_header X-Frame-Options "SAMEORIGIN";
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" $
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";
    add_header Referrer-Policy 'strict-origin';
    }
    
    
    server {
        if ($host = digitalinsa.com) {
            return 301 https://$host$request_uri;
        } # managed by Certbot

        if ($host = www.digitalinsa.com) {
            return 301 https://$host$request_uri;
        }
        
        listen 80;
        server_name digitalinsa.com www.digitalinsa.com;
        return 404; # managed by Certbot
     }

```

## Install Grafana
```
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install -y grafana
sudo /bin/systemctl enable grafana-server
sudo /bin/systemctl start grafana-server
# http://<ip address>:3000 admin admin
```

## Install Docker
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh 

docker container run debian
docker container run -it debian /bin/bash

docker container ls
docker container ls -a
docker container rm c55680af670c

```

## Install Thingsboard Docker
```
docker run -it -p 9090:9090 -p 1883:1883 -p 5683:5683/udp -v ~/.mytb-data:/data -v ~/.mytb-logs:/var/logs/thingsboard --name mytb --restart always thingsboard/tb-postgres

docker run -it -p 9090:9090 -p 1882:1883 -p 5683:5683/udp -v ~/.mytb-data:/data -v ~/.mytb-logs:/var/logs/thingsboard --name HomeHubTB --restart always thingsboard/tb-postgres

$ docker attach mytb

To stop the container:

$ docker stop mytb

To start the container:

$ docker start mytb


Upgrading
In order to update to the latest image, execute the following commands:

$ docker pull thingsboard/tb-postgres
$ docker stop mytb
$ docker run -it -v ~/.mytb-data:/data --rm thingsboard/tb-postgres upgrade-tb.sh
$ docker rm mytb
$ docker run -it -p 9090:9090 -p 1883:1883 -p 5683:5683/udp -v ~/.mytb-data:/data -v ~/.mytb-logs:/var/log/thingsboard --name mytb --restart always thingsboard/tb-postgres
NOTE: replace host's directory ~/.mytb-data with directory used during container creation.
```

