​## Golos.io HowTo Guide
Golosio is blockchain-based microblogging system based on cyberway blockchain. 
Read more on [Golos here](https://wiki.golos.io), on [Cyberway here](https://cyberway.io).

###### tags: `golos` `golosio` `finteh` `fintehdeploy`

## Table of Content
[ToC]

## :memo: Where do I start?

### Step 1: Check hardware requirements
One server'll be "cyberway blockchain node", another - "golos microservices" & "golos web-ui". So, check that you have all of the following:
- [x] Linux server x 2 (note: it's possible to run on one server but it's: a] not recommended b] harder c] will require a lot of resources)
- [x] Hardware Requirements for each server: **4+ CPUs, 16 GB RAM** (the more, the better)
- [x] Server 1 space requirements for full "cyberway blockchain node": **300 + GB** (as-of October 2020)
- [x] Server 2 space requirements for microservices & web-ui: **50 GB**
- [x] Both servers has to have public IP addresses (unless you want to make set-up process even harder)
- [ ] Ideally, both of the servers should have their domain names correctly assigned and SSL configured. This is not a strict requirement, but it's recommended.

### Step 2: Check software requirements
- [x] Operating systems supported and currently verified: <b>Debian 10</b>
- [x] Operating systems supported but will need some additional steps: <b>CentOS 8</b>
- [x] Normal linux packages: bash, curl, fold, head, tail, df, sed, grep, tar, sudo (they should already be installed, maybe with exception of sudo)
- [x] OS that might work as well but ain't fully verified yet: Ubuntu, Fedora, Alpine, ArchLinux

Let's get our hands dirty! :rocket: 

### Step 3: Install additional packages
Before installing new software, it's always considered good to update the system to the current/latest state. Do it with the following:
* ubuntu/debian under normal user: `sudo apt update && sudo apt upgrade`
* fedora/centos under root: `dnf update`
* arch/manjaro under normal user: `sudo pacman -Syyu`
* alpine under root: `apk update && apk upgrade`

:::info
:arrow_right_hook:  If you're going with alpine, it may lack some default tools, like bash, curl and nano. So install them beforehead (`apk add bash curl nano`)
:::

Ok, let's install the required software. We'll need: 
* git >= v2.7, 
* docker v19.03, 
* docker-compose v2.1. 

:::info
:computer: If working under "root" user, you can omit the "sudo" command
:man: If working under normal user, please use sudo
If sudo isn't installed, add it after "git" in the next command
:::

:::info
:exclamation: This software required on all servers (in this HowTo - on two servers). So please go through step 3 for each server.
:::

#### 3.1 Installing git
Let's start with git installation from the OS repository, and then we will install Docker and Docker-compose from Docker's repository as it's usually more fresh there. Here we go:
* ubuntu/debian under normal user: `sudo apt install git`
* fedora/centos under root: `dnf install git`
* arch/manjaro under normal user: `sudo pacman -S git`
* alpine under root: `apk add git`

:::info
:arrow_right: Starting from here, we will assume you are working as normal user, so all commands that requires to be issued from root will start with "sudo". If you're lazy and you're working from root user, but you don't want to remove the sudo from commands, you can run it under root with sudo as well - just make sure that "sudo" package is installed in this case.
:::

Please refer to your package manager documentation if you have other OS or check [this distrowatch article](https://distrowatch.com/dwres.php?resource=package-management). Please be aware that the deployment process has only been tested on OSes shown in Step 2, but it might as well work on others.
 
 #### 3.2 Installing Docker
 
 > If you already have docker installed from system repo's, please remove it first, as it's usually old. For debian/ubuntu, you'll need to run `sudo apt remove docker docker-engine docker.io containerd runc`

Run the docker installation from automated script:
```
curl -fsSL https://get.docker.com | sh
```
This script adds Docker's repositories to your system and installs everything that is required for basic docker on your system. If you don't trust the script, you can as well [do all the steps manually](https://docs.docker.com/engine/install/) or download the script with wget and check it by hand.

:::info
:exclamation: This script does not support alpine linux. To install docker on alpine, please follow the notes from chapter "Alpine 3.12.1 notes".
:::

#### 3.3 Installing Docker-compose

You can either navitage to [this page](https://docs.docker.com/compose/install/) and follow the instructions there, or just run the following script - it gathers 3 variables and downloads recent docker-compose for you:
```bash
current=$(curl https://github.com/docker/compose/releases/latest | awk -F"tag/" '{ print $2 }' | awk -F\" '{ print $1 }')
os=$(uname -s)
arch=$(uname -m)
sudo curl -L "https://github.com/docker/compose/releases/download/$current/docker-compose-$OS-$arch" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```
:::info
:exclamation: On alpine, install docker-compose from the edge repo as well (considering you enabled the edge repo in the previous step): `apk add docker-compose`
:::

:::info
:information_source: Please check that installed Docker & Docker-compose versions are same or higher as required in step 3: `docker --version && docker-compose --version`
:::

If you've done this only on one server, please return to the beginning of the Step 3 and repeat this actions on all of the planned servers.

### Step 4: Cloning & configuring Cyberway blockchain node

For the best options, we need to spin a full Cyberway blockchain node. As noticed in step 1, it will require a lot of hard drive space. Cyberway blockchain works based on NATS message streaming system. [Read more on NATS here](docs.nats.io).

Please log-in into your first server.

:::info
:bulb: From here, we are working on the "Server1".
:::

1. Clone the cyberway.launch repo: 
```git clone https://github.com/cyberway/cyberway.launch```
2. Enter into cloned folder, and then into "nats":
```cd cyberway*/nats```
3. We need to fix the NATS to allow it to use queue of any size. For this, edit the config file `config.conf` in your faviourite text editor, for example, in nano - `nano config.conf`. Uncomment the commented line "#max_bytes" and make it look like this:
```max_bytes: 0```
4. Save the file (in nano - Ctrl+o, Enter). 
5. Exit the text editor (in nano - Ctrl+x)
6. Return to the top cyberway.launch folder: 
```cd ..```

### Step 5: OS-Specific notes

:::info
:bulb: Please don't use outdated and not-supported operating systems. Recommended OSes and their versions are shown in this help document. 
:::

#### Debian (10) and Ubuntu 18.04+
These systems are considered to be the basis for the manual, so no additional steps should be considered for Debian or Ubuntu.

#### CentOS 8

If you're on **CentOS**, you need to fix the firewall for everything to work.
1. To do so, open the firewalld config file first (here is the nano example):
```sudo nano /etc/firewalld/firewalld.conf```
2. Then, change the firewall backend from nftables to this:
```FirewallBackend=iptables```
3. Save the file with Ctrl+o, Enter
4. Exit the nano with Ctrl+x
5. Restart the firewalld service:
```sudo systemctl restart firewalld.service```

> Check that you have NetworkManager configured (DNS was registered):
```nmcli dev show | grep 'IP4. DNS'```

#### Alpine 3.12.1 notes
:::info
:information_source: This hints will probably work for alpine @edge version as well.
:::
* Installation of Docker & Docker-compose has to be done from edge or community system repositories rather then docker auto-install scripts
* To install Docker, Docker-compose, and other stuff, please enable edge repositories for it: `sed -i '/edge/s/^#//' /etc/apk/repositories`
* Don't forget to add docker group so you can use docker sockets: `addgroup $(whoami) docker`
* Adding docker to auto-boot: `rc-update add docker boot`
* Starting docker service: `service docker start`
* After installing docker and docker-compose and doing all the steps from here, for everything to work, please reboot your alpine system - this is the easiest way to make docker work right away.
* Please don't forget to additionally install `bash, curl, nano, grep` and any other tools as alpine does not include them by default (grep is included in alpine, but it's busybox version and it does not work with some of the flags)
* start_full_node.sh script has problem with treating free disk space: on alpine it won't work as-is. If you want to use start_full_node.sh script, please open it with text editor, and replace the row `local avail=$(...)` with the following: 
`local avail=$(df $dir | awk -F" " '{ print $4 }' | tail -1)`
* Alpine runs everything by default with ash rather than bash. So please run start_full_node script like this: `bash start_full_node.sh`. If you want logs to be saved to file you can also use `bash start_full_node.sh > start-logs.txt 2>&1`

#### Fedora 33
Before starting the node, run the following commands and then reboot:
```
sudo dnf install -y grubby && \
  sudo grubby \
  --update-kernel=ALL \
  --args="systemd.unified_cgroup_hierarchy=0"
```
More detailed [here](https://medium.com/nttlabs/cgroup-v2-596d035be4d7)

#### Others
Planned - Add notes here for:
- [ ] Arch Linux
- [ ] // add info here //

### Step 6: Start the Cyberway full blockchain node synchronisation

:::info
:ab: Please consider using a snapshot rather then downloading data directly. It should save you a lot of time. Scroll down to "Uploading from snapshot" section in Appendix if you want to try the snapshot way.
:::

First big step is approaching. In the main cyberway.launch directory, run:
```sudo ./start_full_node.sh```

**This process requires a lot of time, network and processing power**. On a powerful server with good connection, sync of the full node will require about 4 days. On a medium-quality server, it will take up to 8 days. On a crappy server it can take even longer.

:::info
:exclamation: This information is relevant for October 2020. While blockchain grows, it will take even more HDD/SSD space and more time to sync, as well.
:::

To check the current sync state, please run docker logs command:
```sudo docker logs nodeosd --tail 20```

In the output of this command, you'll see current block numbers counting. Please verify the current blockchain state by visiting cyberway blockchain explorer at https://explorer.cyberway.io ("Last block" value).

Before taking a pause for several days, please record some important information, as we gonna need it in the future:
1. Server 1 public IP address:
```ip a```
3. NATs-generated login and password:
```nano /var/lib/cyberway/.env```

Save :inbox_tray: your notes into a safe place.

After several days (or weeks :) of waiting, you can finally proceed to next step!

:::info
:pushpin: Please, don't forcefully kill the containers that is started by *start_full_node.sh* script. Otherwise, you'll probably have to remove everything and start from scratch. To correctly stop the containers, please refer to the chapter "Correctly stopping Cyberway node".
:::

##### Setup CORS policy
On the web, it is important to configure CORS policy, otherwise the browser will block cross-domain requests (because the gate and the frontend itself will be on different domains). 
Install nginx and add the config below by changing the domain to your own
```
sudo apt-get install nginx
```
Configure CORS via nginx:
<details><summary>nginx.conf</summary>

```
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
events {
    worker_connections 1024;
}
http {
  upstream cyberway_node {
     server you_ip:8888;
  }
  server {
    listen you_ip:443 ssl;
    server_name www.example.com;
    add_header 'Access-Control-Allow-Origin' '*' always;
    keepalive_timeout 120;
    keepalive_requests 10000;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    location ~ / {
 
        proxy_pass http://cyberway_node;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-NginX-Proxy true;
        proxy_set_header Connection "upgrade";
        proxy_next_upstream error timeout invalid_header http_500;
        proxy_connect_timeout 2;
        proxy_http_version 1.1;
        proxy_ssl_server_name on;
        proxy_ssl_name "www.example.com";
        #proxy_ssl_name "www. cloudflare. com";
       proxy_read_timeout 86400;
        access_log off;
  }
  
  # uncomment the lines below after you create the ssl certificate
    #include /etc/letsencrypt/options-ssl-nginx.conf;
    #ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
 
  }
}
```

</details>

To create an ssl certificate, you can use Let's Encrypt using [certbot](https://certbot.eff.org/):
```
#Installation on ubuntu/debian
sudo apt-get update 
sudo apt-get install certbot python3-certbot-nginx
```
Check if everything is OK with the config 
```
sudo nginx -t
```
If you get an error, reopen the server block file and check for any typos or missing characters. Once your configuration file syntax is correct, reload Nginx to load the new configuration:
```
sudo systemctl reload nginx
```

Certbot provides a variety of ways to obtain SSL certificates through plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the config whenever necessary. To use this plugin, type the following:
```
sudo certbot --nginx -d your_domain 
```
This runs certbot with the --nginx plugin, using -d to specify the names we’d like the certificate to be valid for.
If this is your first time running certbot, you will be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let’s Encrypt server, then run a challenge to verify that you control the domain you’re requesting a certificate for.

If that’s successful, certbot will ask how you’d like to configure your HTTPS settings.
Your certificates are downloaded, installed, and loaded. Try reloading your website using https:// and notice your browser’s security indicator. It should indicate that the site is properly secured, usually with a green lock icon. If you test your server using the SSL Labs Server Test, it will get an A grade.

### Step 7: Running Golos microservices

Please log-in into your second server.

:::info
:bulb: From here, we are working on the "Server2".
:::

#### 7.1. Clone launch scripts
We'll need the golosio-launching to start the microservices.

1. Clone this repo:
```git clone https://github.com/GolosChain/golosio-launching.git```
2. Enter the cloned folder:
```cd golosio-launching```
3. Copy the example config:
```cp .env.example .env```

#### 7.2 Edit configs
This is very important step. Please do it carefully. We need to edit the configuration files.
1. Open the env-file with nano or other editor:
```nano .env```
2. Replace contents with yours:
```text
CYBERWAY_HTTP_URL=https://<cyberway-node-public-ip>:8888
BLOCKCHAIN_BROADCASTER_URL=nats://user:password@<cyberway-node-public-ip>:4222
GLS_PROVIDER_PUBLIC_KEY=public-key
GLS_PROVIDER_WIF=private-key
GLS_PROVIDER_USERNAME=username
```

* CYBERWAY_HTTP_URL: should point to your IP or DNS name of Server1 that we configured on the previous step. If you'll use IP, please don't forget to change HTTPS to HTTP so it will be http://some.your.ip.address:8888. Also, if you changed port of Cyberway service (default it 8888), you'll need to change it to yours. If you use DNS name, you probably can use both HTTP and HTTPS, if you configured the SSL certificates on your host correctly.
* BLOCKCHAIN_BROADCASTER_URL: you saved notes for the NAT settings (user & password) from the step 6, right? Insert them in here.
* GLS_PROVIDER_PUBLIC_KEY: this should be your GOLOS Public key
* GLS_PROVIDER_WIF: // add info here //
* GLS_PROVIDER_USERNAME: your EOS-style username (you can check it at explorer.cyberway.io)

3. Save your edits With Ctrl+o, Enter.
4. Exit nano with Ctrl+x.

#### 7.3 Run Golosio microservices
Finally, we should be ready now to run the microservices.

```docker-compose up -d --build```

:::info
:-1: Golosio original team tells us, that you [can ignore warnings about old libraries (link)](https://github.com/GolosChain/golosio-launching#%D0%97%D0%B0%D0%BF%D1%83%D1%81%D0%BA). We should probably fix them as soon as we find somebody who'd able to do that.
:::

This step, just like the Cyberway node step 6, will take a long time.

You can check what's happening with the containers and the system meanwhile with this useful commands:
* To check health of all containers: `sudo docker ps`
* To check logs: `sudo docker logs <container_name>`
* Command to see space occupied by all docker-related things: `sudo docker system df -v`
* Additional userful docker flags: `--remove-orphans`
* Useful system commands for debug: 
** `ps axfwwu` for process management
** `df -h` and `du -h --max-depth 1` for free space management
** 'dmesg' for system log


As with the step 6, please check that containers are healthy and return in several days.

### Step 8: Run Golosio Web-UI
This is probably the easiest step unless you do something really weird. Golos Web-UI can run from Third Server, but it's not a strict requirements. In this How-To we will assume that you're running it from the Server2, so the host change isn't required in this case.

Log-in back to your Server2 and check that all the containers finalyzed the sync (check step 7).

Now, let's work on WebUI.

#### 8.1 Clone the repo & edit config
As usual, we need the repo to start the Golos Web-UI.
1. Download the repo
```git clone https://github.com/GolosChain/golos.io.git```
2. Change directory into freshly download one:
```cd golos.io```
3. Copy the reference config file:
```cp .env.example .env```
4. Open it with your favourite text editor (in this example: nano)
```nano .env```
5. Edit it accordingly to your variables:
```text
CYBERWAY_HTTP_URL=https://<node-cyberway-public-ip>:8888
GATE_CONNECT=ws://gate-node:8080
FACADE_CONNECT=http://facade-node:3001
```

So here we have these variables:
* CYBERWAY_HTTP_URL: same as in previous config
* GATE_CONNECT: it can be a name of the docker container
* FACADE_CONNECT: it can be a name of the docker container

#### 8.2 Launch Golosio Web-UI
Let's do the last important step!
`docker-compose up --build`

By default, service is exposed at 3000 port. If you want to edit that, please do the following:
1. Open the .env file with your text editor
2. Edit "DOCKER_CONNECTOR_WEB_PORT=3000" to other port
3. Save the file & exit text editor

---

### Congadulations! You should now have everything up and running!
:fireworks: The end. :fireworks: 

---

## Appendixes

### Uploading from snapshot
Restoring the blockchain state from snapshot is faster and might be as well easier to do. Consider this step as a full replacement for "Step 6: Start the Cyberway full blockchain node synchronisation".

Well, if you decided to use the snapshot, first you'll need to download and unpack snapshot to cyberway.launch/snapshot or another folder.
```
mkdir -p ~/cyberway.launch/snapshot && cd ~/cyberway.launch/snapshot
wget https://download.cyberway.io/snapshot-20201102-v2.1.1.tar
tar -xvf snapshot-20201102-v2.1.1.tar
cd ..
```
Now, we might need to stop the node (if it was running) and re-create volumes (in case if you fiddled with start_full_node script earlier). So in the cyberway.launch folder, run the following commands:
:::info
:information_desk_person: If this is a fresh server and you hadn't touched start_full_node script yet, you may skip next three commands.
:::
```
sudo ./start_full_node.sh down
docker volume rm cyberway-mongodb-data cyberway-nats-data cyberway-nodeos-data cyberway-queue
docker volume create cyberway-mongodb-data cyberway-nats-data cyberway-nodeos-data cyberway-queue
```
`snapshot` is the folder where you unpacked the snapshot - replace it in the command below (three times), if you used a different one.
```
sudo docker run --rm -ti -v `readlink -f snapshot`:/host:ro -v cyberway-nodeos-data:/data:rw cyberway/cyberway:v2.1.1 tar -xPvf /host/nodeos.tar.bz2
sudo docker run --rm -ti -v `readlink -f snapshot`:/host:ro -v cyberway-mongodb-data:/data:rw cyberway/cyberway:v2.1.1 tar -xPvf /host/mongodb.tar.bz2
sudo docker run --rm -ti -v `readlink -f snapshot`:/host:ro -v cyberway-nats-data:/data:rw cyberway/cyberway:v2.1.1 tar -xPvf /host/nats.tar.bz2
sudo ./start_full_node.sh up
```

:::info
:information_source: On alpine, you will need to run `bash ./start_full_node.sh up`, because bash isn't default script interpreter
:::

Now, please continue where you left off (probably, your next step will be "Step 7: Running Golos microservices").

### Additional notes
"Cyberway-notifier" service doesen't put timestamps in it's logs by default according to @vikxx. So if the log contains error in the end of it, it does not mean, that error is still relevant.

### Opening ports
In case if your host is behind a router, please open ports and direct them into your local host. All ports are TCP. Port list follows:
* cyberway node: 8888, 9876, 9216, 4222, 8222, 27018, 7777
* golos io microservices: 8080, 3001
* web-ui: 443 and/or 80

### List of Cyberway seed nodes
Please check the list here:
* https://github.com/cyberway/cyberway.launch/blob/master/seednodes

It's also possible to add the seed nodes inside of the "config.ini" file by specifying multiple `p2p-peer-address` variables in that (config.ini) file:
```
p2p-peer-address = seed.cyberval.net:9876
p2p-peer-address = cyber1.lexai.host:9876
... etc ...
```

### Fixing NATs issues
If something happend during NATs sync, you can do the following:
1. Follow all 3 steps from "Stopping Cyberway Full Node" (previous step)
2. Remove NATs volume:
```sudo docker volume rm cyberway-nats-data```
3. Launch node with replay argument:
```sudo env EXTRA_NODEOS_ARGS=" --replay-blockchain --fix-reversible-blocks " ./start_full_node.sh```
:::info
:information_source: if you are under root don't use sudo
:::

## Correctly stopping the services

:::info
:information_source: Assuming by default that all folders (cyberway.launch, golosio-launching) was cloned to the user's home directory or just your current directory.
:::

### Stopping Cyberway Full Node
1. Stop the nodeosd first, with 2-minute delay:
```sudo docker stop -t 120 nodeosd```
2. After two minutes, check that everything stopped:
```sudo docker logs --tail 10 -f nodeosd```
3. Stopping node services (assuming cyberway.launch is at your ~home):
```cd cyberway.launch && sudo ./start_full_node.sh down```

### Stopping Golosio Microservices
:::info
Normal stopping of Microservices does not include full resync
:::
1. Enter the project directory:
```cd golosio-launching```
2. Stop the services:
```docker-compose down```

### Stopping Golosio Microservices with full resync
:::info
To include full resync of microservices we have to remove docker volumes (-v flag, so) 
:::
1. Enter the project directory:
```cd golosio-launching```
2. Stop the services:
```docker-compose down -v```

### Stopping Golosio Web-GUI
1. Enter the project directory:
```cd golos.io```
2. Stop the web-ui:
```docker-compose down ```

## Correctly restarting services after you stopped them

### Restarting Cyberway Full Node
1. First, check, that you've stopped the services like in "Stopping Cyberway Full Node"
2. Enter the folder:
```cd cyberway.launch```
3. Restart the full node:
```sudo ./start_full_node.sh up```

### Restarting Golosio Microservices
1. First, check, that you've stopped the services like in "Stopping Golosio Microservices"
2. Enter the project directory:
```cd golosio-launching```
3. Restart the services:
```docker-compose up -d --build```

### Restarting Golosio Web-GUI
1. Enter the project directory:
```cd golos.io```
2. Run the web-ui:
```docker-compose up -d --build```

## Changelog: latest additions
* [09-11-2020] Alpine instructions and notes
* [November 2020] Additional notes on CentOS procedure
* [October 2020] Staring of this documentation with Debian 10 example

## Contributors to this documentation

| Role | Telegram | Golos | Github |
| - | - | - | - |
| Creator | @fakesnowden | @sxiii | @sxiii 
| Based-on | @a_falaleev | ? | @afalaleev
| Based-on | @b1acksun | ? | @b1acksun
| Help-from | @rubynzon | ? | @Moisey777
| Help-from | @vikxx | @vik | @vikxx
| Financial-help | @finteh | @finteh | @finteh

### How to improve these docs further
```sequence
Alice->Bob: Hello Bob, are the docs perfect?
Note right of Bob: Bob thinks
Bob-->Alice: I am checking them now!
Note left of Alice: Alice responds
Alice->Bob: Why haven't you checked them before?
```

## Is it possible to help?
Yes, for sure! We need the following help:
* Testing this documentation on the same (Debian, CentOS) and others (Ubuntu, Fedora, Alpine, Arch, etc.) distros
* Adding important information and notes
* Running your own nodes to help the Cyberway blockchain as well as microservices and GUI for Golos
* Coding help, code rewriting and refreshing/modernisation
* Translation help
* UI/UX design help
* Project architecture discussion
* Add asciinema explaination video
* Add youtube teach guide video
* Push it to github

Add to [fincubator group](https://t.me/fincubator) in telegram and do golos better!
