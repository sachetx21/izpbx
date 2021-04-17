# Name
[<img src="https://www.initzero.it/images/initzero-logo-izpbx-48x48.png">](https://www.initzero.it) izPBX Cloud Native VoIP Telephony System

# Description
izPBX is a Turnkey Cloud Native VoIP Telephony System powered by Asterisk Engine and FreePBX Management GUI

# Supported tags

## Production Branch: (Asterisk 18 + FreePBX 15)
* `latest`, `18`, `18.15`, `18.15.X`, `18.15.X-BUILD`, `18.15.X-COMMIT`

## Legacy Production Branch: (Asterisk 16 + FreePBX 15)
* `0`, `0.9`, `0.9.X`, `0.9.X-BUILD`, `0.9.X-COMMIT`

## Development Branches:
* Asterisk 18 LTS + FreePBX 15: `dev-18`, `dev-18.X`, `dev-18.X.X-BUILD`, `dev-18.X.X-COMMIT`
* Asterisk 16 LTS + FreePBX 15: `dev-16`, `dev-16.X`, `dev-16.X.X-BUILD`, `dev-16.X.X-COMMIT`

## Version notes:
Tags format: **Z.Y.X-[BUILD|COMMIT]**

where:  
  * **Z** = Asterisk release (PBX Engine)
  * **Y** = FreePBX release (PBX GUI)
  * **X** = izPBX release (PBX Distribution)
  * **BUILD** = Build number | **COMMIT** = GIT commit ID

Look into project [Tags](https://hub.docker.com/r/izdock/izpbx-asterisk/tags) page to discover the latest versions

# Dockerfile
- https://github.com/ugoviti/izdock-izpbx/blob/master/izpbx-asterisk/Dockerfile

# Features
- Fast initial bootstrap to deploy a full features PBX system (60 secs install time from zero to a running turnkey PBX system)
- Built-in PBX Engine based on Asterisk® project (compiled from scratch)
- Built-in WEB Management GUI based on FreePBX® project (with default predownloaded modules for quicker initial deploy)
- No vendor lock-in, you can migrare to and away izPBX simply importing/exporting FreePBX Backups
- Based on Linux CentOS 8 64bit OS
- Small container image footprint
- Multi-Tenant PBX System Support (look into **Advanced Production Configuration Examples** section)
- Automatic Remote XML PhoneBook support for compatible VoIP Phones
- Persistent storage mode for configuration and not volatile data
- Fail2ban as security monitor to block SIP and HTTP brute force attacks
- FOP2 Operator Panel
- Integrated Asterisk Zabbix agent for active health monitoring
- Misc `izpbx-*` scripts (like `izpbx-callstats`)
- `izsynth` utility - TTS/Text To Speech synthesizer, background music overlay assembler and audio file converter for PBX and Home Automation Systems
- `tcpdump` and `sngrep` utility to debug VoIP calls
- supervisord as services management with monitoring and automatic restart when services fail
- postfix MTA daemon for sending mails (notifications, voicemails and FAXes)
- Integrated cron daemon for running scheduled tasks
- TFTP and DHCP server powered by DNSMasq for autoprovisioning VoIP Phones
- Apache 2.4 and PHP 7.3 (mpm_prefork+mod_php configuration mode)
- Automatic Let's Encrypt HTTPS certificate management for exposed PBXs to internet
- Custom commercial SSL Certificates support
- Logrotating of service logs
- All configurations made via single central `.env` file
- Many customizable variables to use (look inside `default.env` file)
- Only two containers setup: (**Antipattern container design** but needed by the FreePBX ecosystem to works)
  - **izpbx** (izpbx-asterisk container: Asterisk Engine + FreePBX Frontend + others services)
  - **izpbx-db** (mariadb container: Database Backend)

# Screenshots
#### izPBX Dashboard (FreePBX):
![izpbx-dashboard](https://raw.githubusercontent.com/ugoviti/izdock-izpbx/master/screenshots/izpbx-dashboard.png)

#### izPBX Operator Panel (FOP2):
![izpbx-izpbx-operator-panel](https://raw.githubusercontent.com/ugoviti/izdock-izpbx/master/screenshots/izpbx-operator-panel.png)

#### izPBX Monitoring Dashboard (Zabbix):
![izpbx-zabbix-dashboard](https://raw.githubusercontent.com/ugoviti/izdock-izpbx/master/screenshots/izpbx-zabbix-dashboard.png)

#### izPBX CLI (Asterisk):
![izpbx-console](https://raw.githubusercontent.com/ugoviti/izdock-izpbx/master/screenshots/izpbx-cli.png)

# Targets of this project
On-Premise, fast, automatic and repeatable deploy of PBX systems.  
by default `network_mode: host` is used, so the PBX network is esposed directly in the host interface (no internal container network is used), so the default UDP RTP port range can be from `10000` to `20000`.  
If you plan to disable `network_mode: host`, tune the port range (forwarding 10000 ports with the docker stack make high cpu usage and longer startup times), for example for 50 concurrent calls:  
`APP_PORT_RTP_START=10000`  
`APP_PORT_RTP_END=10200`  
for best security, fine-tune the ports range based on your needs by not using standard port ranges!  

# Limits of this project
- Deploy 1 izPBX for every external IP or single VM. No multi containers setup works out of the box right now (caused by technical limits of how Docker and SIP UDP RTP traffic works)
- Container Antipattern Design (FreePBX was not designed to run as containerized app, and its ecosystem requires numerous modules to function, and the FreePBX modules updates will managed by FreePBX Admin Modules Pages itself not by izPBX container updates)
  
# Deploy izPBX
Using **docker-compose** is the suggested method:

- Install your prefered Linux OS into VM or Baremetal Server

- Install Docker Runtime and docker-compose utility for your Operating System from https://www.docker.com/get-started
  - CentOS 8 Quick&Dirty commands (skip if you use other distribution):
```
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce -y
eval sudo curl -L "$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep browser_download_url | grep "docker-compose-$(uname -s)-$(uname -m)\"" | awk '{print $2}')" -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose
sudo systemctl enable --now docker
```

- Clone GIT repository or download latest release from: https://github.com/ugoviti/izdock-izpbx/releases and unpack it into a directory (ex. `/opt/izpbx`):
  - `git clone https://github.com/ugoviti/izdock-izpbx.git /opt/izpbx`
  - `cd /opt/izpbx`

- Checkout into latest official release:
  - `git checkout refs/tags/$(git tag | sort --version-sort | tail -1)`

- Copy default configuration file `default.env` into `.env`:
  - `cp default.env .env`

- Customize `.env` variables, specially the security section of default passwords (look bellow for a full variables list):
  - `vim .env`

- Deploy and start izpbx using docker-compose command:
  - `docker-compose up -d`

- Wait the pull to finish (~60 seconds with fast lines) and point your web browser to the IP address of your docker host and follow initial setup guide

Note: by default, to correctly handle SIP NAT and SIP-RTP UDP traffic, the izpbx container will use the `network_mode: host`, so the izpbx container will be exposed directly to the outside network without using docker internal network range (**network_mode: host** will prevent multiple izpbx containers from running inside the same host).  
Modify docker-compose.yml and comment `#network_mode: host` if you want run multiple izpbx containers in the same host (not tested. there will be problems with RTP traffic).

If you want test izPBX without using docker-compose, you can use the following docker commands:

Start MySQL:  
`docker run --rm -ti -v ./data/db:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=CHANGEM3 -e MYSQL_PASSWORD=CHANGEM3 --name izpbx-db mariadb:10.4`

Start izPBX:  
`docker run --rm -ti --network=host --privileged --cap-add=NET_ADMIN -v ./data/izpbx:/data -e MYSQL_ROOT_PASSWORD=CHANGEM3 -e MYSQL_PASSWORD=CHANGEM3 -e MYSQL_SERVER=127.0.0.1 -e MYSQL_DATABASE=asterisk -e MYSQL_USER=asterisk -e APP_DATA=/data --name izpbx izpbx-asterisk:latest`

# Upgrade izPBX

1. Upgrade the version of izpbx by downloading a new tgz release, or changing image tag into **docker-compose.yml** file (from git releases page, verify if upstream docker compose was updated), or if you cloned directly from GIT, use the following commands as quick method:
```
cd /opt/izpbx
git pull
git checkout refs/tags/$(git tag | sort --version-sort | tail -1)
```

2. Upgrade the **izpbx** deploy with:  
(NB. **First** verify if `docker-compose.yml` and `default.env` was updated a make the same changes in your `.env` file)
```
docker-compose pull
docker-compose up -d
```

3. If the mariadb database version was changed, rememeber to update tables schema with command  
  `source .env ; docker exec -it izpbx-db mysql_upgrade -u root -p$MYSQL_ROOT_PASSWORD`

4. Open FreePBX Web URL and verify if exist any modules updates from FreePBX Menù: **Admin-->Modules Admin: Check Online**

That's all

### FreePBX upgrade path to a major release
FreePBX will be installed into persistent data dir on initial deploy only (when no installations already exist).

Successive container updates will not upgrade the FreePBX Framework (only Asterisk engine will be updated).  
After initial deploy, upgrading FreePBX Core and Modules and Major Release (es. from 15.x to 16.x) is possible **only** via **official FreePBX upgrade method**:
  - FreePBX Menù: **Admin-->Modules Admin: Check Online** select **FreePBX Upgrader**

Recap: only Asterisk core engine will be updated on container image update. FreePBX will be updated only via Modules Update Menu.

# Advanced Production Configuration Examples

#### Multi-Tenant VoIP PBX with dedicated Databases

##### Objective
- Run many izPBX instances into single docker host (you must allocate a private static IP for every izPBX backend/frontend)
- Dedicated Database for every izPBX instances

##### Configuration
Create a directory where you want deploy izpbx data and create `docker-compose.yml` and `.env` files:

Example:
```
mkdir yourgreatpbx
cd yourgreatpbx
vim docker-compose.yml
```

NOTE:
- Modify `docker-compose.yml` and change according to your environment needs:
  - `parent:` (must be specified your ethernet card)
  - `subnet:` (must match you intranet network range)
  - `ipv4_address:` (every izPBX frontend will must to have a different external IP)
- Rembember to modify every `PBXNAME.env` file and set different variables for `MYSQL` (for best security use a different password for every instance), example:
  - `MYSQL_SERVER=db`
  - so on...

```yaml
version: '3'

networks:
  izpbx-int:
    driver: bridge
  izpbx-ext:
    driver: macvlan
    driver_opts:
      parent: enp0s13f0u3u1u3
    ipam:
      config:
      - subnet: 10.1.1.0/24
        
services:
  izpbx:
    #hostname: ${APP_FQDN}
    image: izdock/izpbx-asterisk:18.15.10
    restart: unless-stopped
    depends_on:
    - db
    env_file:
    - .env
    volumes:
    - /etc/localtime:/etc/localtime:ro
    - ./data/izpbx:/data
    cap_add:
    - SYS_ADMIN
    - NET_ADMIN
    privileged: true
    networks:
     izpbx-int:
     izpbx-ext:
       ipv4_address: 10.1.1.221

  db:
    image: mariadb:10.5.9
    ## WARNING: if you upgrade image tag enter the container and run mysql_upgrade:
    ## source .env ; docker exec -it izpbx-db mysql_upgrade -u root -p$MYSQL_ROOT_PASSWORD
    command: --sql-mode=ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
    restart: unless-stopped
    env_file:
    - .env
    environment:
    - MYSQL_ROOT_PASSWORD
    - MYSQL_DATABASE
    - MYSQL_USER
    - MYSQL_PASSWORD
    ## database configurations 
    volumes:
    - /etc/localtime:/etc/localtime:ro
    - ./data/db:/var/lib/mysql
    networks:
     izpbx-int:
```

Repeat the procedure for every izPBX you want deploy. Remember to create a dedicated directory for every izpbx deploy.

##### Deploy
Enter the directory containig configuration files and run:
- `docker-compose up -d`

#### Multi-Tenant VoIP PBX with shared global Database and single docker-compose.yml file

##### Objective
- Run many izPBX instances into single docker host (you must allocate a private static IP for every izPBX backend/frontend)
- Single Shared Database used by all izPBX instances

##### Configuration
Create a directory where you want deploy all izpbx data and create `docker-compose.yml` and a `PBXNAME.env` file for every izpbx deploy:

Example:
```
mkdir izpbx
cd izpbx
vim docker-compose.yml
vim izpbx1.env
vim izpbx2.env
vim izpbx3.env
```
etc...

NOTE:
- Modify `docker-compose.yml` according to your environment needs, changing:
  - `parent:` (must be specified your ethernet card)
  - `subnet:` (must match you intranet network range)
  - `ipv4_address:` (every izPBX frontend will must to have a different external IP)
- Rembember to modify every `PBXNAME.env` file and set different variables for `MYSQL` (for best security use a different password for every deploy), example:
  - `MYSQL_SERVER=db` (all deployes will use the same db name)
  - `MYSQL_DATABASE=izpbx1_asterisk`
  - `MYSQL_DATABASE_CDR=izpbx1_asteriskcdrdb`
  - `MYSQL_USER=izpbx1_asterisk`
  - `MYSQL_PASSWORD=izpbx1_AsteriskPasswordV3ryS3cur3`
  - so on...

```yaml
version: '3'

networks:
  izpbx-int:
    driver: bridge
  izpbx-ext:
    driver: macvlan
    driver_opts:
      parent: enp0s13f0u3u1u3
    ipam:
      config:
      - subnet: 10.1.1.0/24
       
services:
  db:
    image: mariadb:10.5.9
    ## WARNING: if you upgrade image tag enter the container and run mysql_upgrade:
    ## source .env ; docker exec -it izpbx-db mysql_upgrade -u root -p$MYSQL_ROOT_PASSWORD
    command: --sql-mode=ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
    restart: unless-stopped
    env_file:
    - db.env
    environment:
    - MYSQL_ROOT_PASSWORD
    - MYSQL_DATABASE
    - MYSQL_USER
    - MYSQL_PASSWORD
    ## database configurations 
    volumes:
    - /etc/localtime:/etc/localtime:ro
    - ./data/db:/var/lib/mysql
    networks:
     izpbx-int:

  izpbx1:
    #hostname: ${APP_FQDN}
    #image: izdock/izpbx-asterisk:18.15.10
    image: izpbx-asterisk:dev-18
    restart: unless-stopped
    depends_on:
    - db
    env_file:
    - izpbx1.env
    volumes:
    - /etc/localtime:/etc/localtime:ro
    - ./data/izpbx1:/data
    cap_add:
    - SYS_ADMIN
    - NET_ADMIN
    privileged: true
    networks:
     izpbx-int:
     izpbx-ext:
       ipv4_address: 10.1.1.221

  izpbx2:
    #hostname: ${APP_FQDN}
    #image: izdock/izpbx-asterisk:18.15.10
    image: izpbx-asterisk:dev-18
    restart: unless-stopped
    depends_on:
    - db
    env_file:
    - izpbx2.env
    volumes:
    - /etc/localtime:/etc/localtime:ro
    - ./data/izpbx2:/data
    cap_add:
    - SYS_ADMIN
    - NET_ADMIN
    privileged: true
    networks:
     izpbx-int:
     izpbx-ext:
       ipv4_address: 10.1.1.222

  izpbx3:
    #hostname: ${APP_FQDN}
    #image: izdock/izpbx-asterisk:18.15.10
    image: izpbx-asterisk:dev-18
    restart: unless-stopped
    depends_on:
    - db
    env_file:
    - izpbx3.env
    volumes:
    - /etc/localtime:/etc/localtime:ro
    - ./data/izpbx3:/data
    cap_add:
    - SYS_ADMIN
    - NET_ADMIN
    privileged: true
    networks:
     izpbx-int:
     izpbx-ext:
       ipv4_address: 10.1.1.223
```

##### Deploy
Enter the directory containig configuration files and run:
- `docker-compose up -d`

# Services Management

### Command to restart Asterisk PBX
`docker restart izpbx`

### Command to restart MariaDB Database
`docker restart izpbx-db`

### If you want restart single services inside `izpbx` container
Enter the container:  
`docker exec -it izpbx bash`

Restart izpbx service (Asterisk Engine):  
`supervisorctl restart izpbx`

To restart others available services use `supervisorctl restart SERVICE`

Available services:  
  - `asterisk`
  - `cron`
  - `fail2ban`
  - `fop2`
  - `httpd`
  - `izpbx`
  - `tftpd`
  - `postfix`
  - `zabbix-agent`

# Tested systems and host compatibility
Tested Docker Runtime:
  - moby-engine 19.03
  - docker-ce 19.03
  - docker-compose 1.25

Tested Host Operating Systems:
  - CentOS 6/7/8
  - Fedora Core >30
  - Debian 10
  - Ubuntu 20.04

# Environment default variables
```bash
### Persistent data management
## enable persistent data storage (comment if you want disable persistence of data) (default: /data)
APP_DATA=/data


### Database
## WARNING: izPBX security passwords... please change the default
MYSQL_ROOT_PASSWORD=CHANGEM3
MYSQL_PASSWORD=CHANGEM3

## WARNING: if docker-compose is configured with "network_mode: bridge" then use "MYSQL_SERVER=db"
## WARNING: if docker-compose is configured with "network_mode: bridge" then use "MYSQL_SERVER=127.0.0.1" or the address of a remote database server
#MYSQL_SERVER=db
MYSQL_SERVER=127.0.0.1
MYSQL_DATABASE=asterisk
MYSQL_DATABASE_CDR=asteriskcdrdb
MYSQL_USER=asterisk


### Email addreses and SMTP smarthost
## outgoing mails will set as From as: (default: izpbx@localhost.localdomain)
#SMTP_MAIL_FROM=izpbx@example.com

## outgoing mails will to send notifications, like cron, fail2ban, etc... (default: root@localhost.localdomain)
#SMTP_MAIL_TO=admin@example.com

## specify DNS name or IP address for the SMTP RelayHost (default: none)
#SMTP_RELAYHOST=[smtp.example.com]:587
#SMTP_RELAYHOST_USERNAME=yourusername
#SMTP_RELAYHOST_PASSWORD=yoursecurepassword
#SMTP_STARTTLS=true
#SMTP_ALLOWED_SENDER_DOMAINS=127.0.0.0/8 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16
#SMTP_MESSAGE_SIZE_LIMIT=67108864


### Webserver and HTTP/HTTPS
## enable if the pbx is exposed to internet and want autoconfigure virtualhosting based on the following FQDN (default: none)
#APP_FQDN=izpbx.example.com

## enable https protocols (default: true)
## place your custom SSL certs in $APP_DATA/etc/pki/izpbx (use filename 'izpbx.crt' for public key and 'izpbx.key' for the private)
## by default izpbx will use a self-signed certificate
#HTTPD_HTTPS_ENABLED=true

## redirect unencrypted http connetions to https (default: false)
#HTTPD_REDIRECT_HTTP_TO_HTTPS=false

## auto generate Let's Encrypt SSL certificates if the pbx is exposed to internet and want enable https protocol (default: false)
## To use LETSENCRYPT make sure SMTP_MAIL_TO, APP_FQDN above are set to correct values
#LETSENCRYPT_ENABLED=true
#LETSENCRYPT_COUNTRY_CODE=IT
#LETSENCRYPT_COUNTRY_STATE=Rome

## by default everyone can connect to HTTP/HTTPS WEB interface, comment out to restrict the access and enhance the security (default: 0.0.0.0/0)
#HTTPD_ALLOW_FROM=127.0.0.0/8 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16


### phpMyAdmin
#PMA_ALIAS=/admin/pma
#PMA_ALLOW_FROM=127.0.0.0/8 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16


### FOP2 configuration (https://www.fop2.com/docs/)
#FOP2_LICENSE_NAME=<put here your corporation name>
#FOP2_LICENSE_CODE=<put here your license code>
#FOP2_LICENSE_IFACE=docker0

## the following variables are not mandatory, you can leave commented (FOP2_AMI_PASSWORD will be a random hash)
#FOP2_AMI_HOST=localhost
#FOP2_AMI_PORT=5038
#FOP2_AMI_USERNAME=admin
#FOP2_AMI_PASSWORD=amp111


### Zabbix Network Monitoring
## for automatic discovery of HOSTNAME leave ZABBIX_HOSTNAME commented
#ZABBIX_SERVER=zabbixserver.example.com
#ZABBIX_HOSTNAME=izpbx.example.com
#ZABBIX_HOSTMETADATA=izPBX CHANGEM3WithAS3cur3HA$H


### Fail2ban 
## format: FAIL2BAN_SECTION_KEY=VALUE
## by default izpbx will will use: FAIL2BAN_DEFAULT_SENDER=$SMTP_MAIL_FROM and FAIL2BAN_DEFAULT_DESTEMAIL=$SMTP_MAIL_TO, anyway you can override it bellow
FAIL2BAN_ENABLED=true
FAIL2BAN_ASTERISK_ENABLED=true
#FAIL2BAN_ASTERISK_LOGPATH=/var/log/asterisk/security
#FAIL2BAN_DEFAULT_SENDER=fail2ban@example.com
#FAIL2BAN_DEFAULT_DESTEMAIL=security@example.com
FAIL2BAN_DEFAULT_IGNOREIP=127.0.0.0/8 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16
FAIL2BAN_DEFAULT_BANTIME=300
FAIL2BAN_DEFAULT_FINDTIME=3600
FAIL2BAN_DEFAULT_MAXRETRY=10
FAIL2BAN_RECIDIVE_ENABLED=true
FAIL2BAN_RECIDIVE_BANTIME=1814400
FAIL2BAN_RECIDIVE_FINDTIME=15552000
FAIL2BAN_RECIDIVE_MAXRETRY=10


### FreePBX Advanced Settings
## prefix every FreePBX internal variable with FREEPBX_
## modules enabled on first startup
#FREEPBX_MODULES_EXTRA=soundlang callrecording cdr conferences customappsreg featurecodeadmin infoservices logfiles music manager arimanager filestore recordings announcement asteriskinfo backup callforward callwaiting daynight calendar certman cidlookup contactmanager donotdisturb fax findmefollow iaxsettings miscapps miscdests ivr parking phonebook presencestate printextensions queues cel timeconditions pm2
FREEPBX_FREEPBX_SYSTEM_IDENT=izPBX
FREEPBX_AS_DISPLAY_READONLY_SETTINGS=1
FREEPBX_AS_OVERRIDE_READONLY=1
FREEPBX_ENABLECW=0
FREEPBX_TONEZONE=it
FREEPBX_PHPTIMEZONE=Europe/Rome
#FREEPBX_BRAND_IMAGE_TANGO_LEFT=images/tango.png
#FREEPBX_BRAND_IMAGE_FREEPBX_FOOT=images/freepbx_small.png
#FREEPBX_BRAND_IMAGE_SPONSOR_FOOT=images/sangoma-horizontal_thumb.png
#FREEPBX_BRAND_FREEPBX_ALT_LEFT=FreePBX
#FREEPBX_BRAND_FREEPBX_ALT_FOOT=FreePBX®
#FREEPBX_BRAND_SPONSOR_ALT_FOOT=www.sangoma.com
#FREEPBX_BRAND_IMAGE_FREEPBX_LINK_LEFT=http://www.freepbx.org
#FREEPBX_BRAND_IMAGE_FREEPBX_LINK_FOOT=http://www.freepbx.org
#FREEPBX_BRAND_IMAGE_SPONSOR_LINK_FOOT=http://www.sangoma.com
#FREEPBX_RSSFEEDS=

## WORKAROUND @20200322 https://issues.freepbx.org/browse/FREEPBX-20559 : fwconsole setting SIGNATURECHECK 0
FREEPBX_SIGNATURECHECK=0

### PhoneBook Settings
## PhoneBook server address used by VoiP Phones.
## You can specify IP or DNS name. If empty, will be used in order: 'http://$APP_FQDN' or 'http://PBXIP'. (default: null)
#PHONEBOOK_ADDRESS=https://izpbx.example.com

### DHCP/NTP/TFTP Server
#DHCP_DOMAIN=izpbx.local
#DHCP_POOL_START=10.1.1.10
#DHCP_POOL_END=10.1.1.250
#DHCP_POOL_LEASE=72h
## leave commented to use docker container ip address
#DHCP_DNS=10.1.1.1
#DHCP_GW=10.1.1.1
#DHCP_NTP=10.1.1.1


### Container Network Ports
## webserver and freepbx ports
APP_PORT_HTTP=80
APP_PORT_HTTPS=443
# asterisk ports
APP_PORT_PJSIP=5160
APP_PORT_SIP=5060
APP_PORT_IAX=4569
APP_PORT_AMI=8088
## WARNING: tune the APP_PORT_RTP_END to a lower value (ex. 10200) if 'network_mode: host' is not used
APP_PORT_RTP_START=10000
APP_PORT_RTP_END=20000
# database port
APP_PORT_MYSQL=3306
# other services ports
APP_PORT_DHCP=67
APP_PORT_TFTP=69
APP_PORT_FOP2=4445
APP_PORT_ZABBIX=10050


### Container Services
POSTFIX_ENABLED=true
CRON_ENABLED=true
HTTPD_ENABLED=true
IZPBX_ENABLED=true
FAIL2BAN_ENABLED=true
#DHCP_ENABLED=true
#TFTP_ENABLED=true
#FOP2_ENABLED=true
#ZABBIX_ENABLED=true
#PMA_ENABLED=true
PHONEBOOK_ENABLED=true
```

# Zabbix Agent Configuration
Consult official repository page for installation and configuration of Asterisk Zabbix Template in you Zabbix Server:
- https://github.com/ugoviti/zabbix-templates/tree/master/asterisk

# FreePBX Configuration Best Practices
* **Settings-->Advanced Settings**
  * CW Enabled by Default: **NO**
  * Country Indication Tones: **Italy**
  * Ringtime Default: **60 seconds**
  * Speaking Clock Time Format: **24H**
  * PHP Timezone: **Europe/Rome**
  
* **Settings-->Asterisk Logfile Settings**
  * Security Settings-->Allow Anonymous Inbound SIP Calls: **No**
  * Security Settings-->Allow SIP Guests: **No**

* **Settings-->Asterisk SIP Settings**
  * File Name: **security**
  * Security: **ON** (all others OFF)
  
* **Settings-->Filestore-->Local**
  * Path Name: **Local Storage**
  * Path: **__ASTSPOOLDIR__/backup**

* **Admin-->Backup & Restore**
  * Basic Information-->Backup Name: **Daily Backup**
  * Notifications-->Email Type: **Failure**
  * Storage-->Storage Location: **Local Storage**
  * Schedule and Maintinence-->Enabled: **Yes**
  * Schedule and Maintinence-->Scheduling: Every: **Day** Minute: **00** Hour: **00**
  * Maintinence-->Delete After Runs: **0**
  * Maintinence-->Delete After Days: **14**

* **Admin-->Contact Manager**
  * External
    * Add New Group
      * Name: **PhoneBook**
      * Type: **External**
  
* **Admin-->Caller ID Lookup Sources**
  * Source Description: **ContactManager**
  * Source type: **Contact Manager**
  * Cache Results: **No**
  * Contact Manager Group(s): **All selected**
  
* **Admin-->Sound Languages-->Setttings**
  * Global Language: **Italian**

# Configuring VoIP XML PhoneBook Lookup
NOTE: Tested on Yealink Phones

- Configure **Contact Manager** as reported above (the Contact Manager GroupName be MUST named **PhoneBook** otherwise doesn't works by default)

## Option 1: PhoneBook Menu
- Open VoIP Phone GUI (Yealink Phone GUI):
  - **Directory-->Remote Phone Book**
    - Index 1 (URL for XML Menu)
      - RemoteURL: **http://PBX_ADDRESS/pb**
      - Display Name: **PhoneBook**

## Option 2: Define every PhoneBook you want to use
- Open VoIP Phone GUI (Yealink Phone GUI):
  - **Directory-->Remote Phone Book**
    - Index 1 (URL for Extensions PhoneBook)
      - RemoteURL: **http://PBX_ADDRESS/pb/yealink/ext**
      - Display Name: **Extensions**
    - Index 2 (URL for Shared PhoneBook)
      - RemoteURL: **http://PBX_ADDRESS/pb/yealink/cm**
      - Display Name: **Shared Phone Book**
      
# FAQ / Trobleshooting
- FreePBX is slow to reload (https://issues.freepbx.org/browse/FREEPBX-20559)
  - As temporary WORKAROUND enter into izpbx container shell and run:  
    `docker exec -it izpbx bash`  
    `fwconsole setting SIGNATURECHECK 0`

- Factory Reset / Start from scratch a clean configurations (WARNING! you persistent storage will be wiped!):
  - `docker-compose down`
  - `rm -rf data`
  - `docker-compose up -d`
    
# TODO / Future Development by priority
- Kubernetes deploy via Helm Chart (major problems for RTP UDP ports range... needs further investigation, no valid solutions right now)
- Hylafax+ Server + IAXModem (used for sending FAXes. Receiving FAXes via mail is already possibile using FreePBX FAX Module)
- macOS host support? (edit docker-compose.yml and comment localtime volume?)
- Windows host support (need to use docker volume instead local directory path?)

# Quick reference
- **Developed and maintained by**:
  [Ugo Viti](https://github.com/ugoviti/izdock-izpbx) @ InitZero S.r.l.

- **Where to file issues**:
  [https://github.com/ugoviti/izdock-izpbx/issues](https://github.com/ugoviti/izdock-izpbx/issues)

- **Where to get commercial help**:
  email: [support@initzero.it](mailto:support@initzero.it) - web: [InitZero Support](https://www.initzero.it/)
  
- **Supported architectures**:
  [`amd64`]

- **Supported Docker versions**:
  [the latest release](https://github.com/docker/docker-ce/releases/latest) (down to 1.6 on a best-effort basis)

- **License**:
  [GPL v3](https://github.com/ugoviti/izdock-izpbx/blob/master/LICENSE)
