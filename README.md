

# Installation of Rocket chat Aws:
## Automated Method
### Build the required for automate deploy:
#### Gitsetup

* First step is to set up  create a git repository of what you want to deploy.
* So the git repository should contain the lisence.txt,appspec.yml, and the contentfolders. 
* After setting up these files also make an zip file of these and upload to the github of yours.
* So accordingly
__git clone https://github.com/RocketChat/Docker.Official.Image into your repository.__
* So your reository must look like
* /ra3426vi-s
*   └-- MyDemoRepo
*       │-- appspec.yml
*       │-- index.html
*       │-- LICENSE.txt(since ubuntu machine needs liscences)
*       └-- scripts
*           │-- install_dependencies
*           │-- after_install
*           └-- start_server

* Next, to launch the deployment from codepipeline with codedeploy  

* Now lets see  and understand how appspec.yml
* version: 0.0
* os: linux
* files:
*  - source: /index.html
*    destination: bin/var/data
* hooks:
*  BeforeInstall:
*    - location: scripts/install_dependencies
*      timeout: 300
*      runas: root
*  AfterInstall:      
*    - location: scripts/after_install
*      timeout: 300
*      runas: root
* ApplicationStart:      
*    - location: scripts/start_server
*      timeout: 300
*      runas: root


* -source says what file i have to take from github and where ishould depoy it if the deployment path does’nt exist it will create the directory and copies the file  .First checking source and destination directories if it doesn’t match then it creates it.

* -hooks :Hooks allow you to specify scripts to run during each of the CodeDeploy events. There are five events you can listen to:

* ApplicationStop, which occurs at the start of a deployment operation.
* BeforeInstall, which occurs before any files specified in the appspec.yml are deployed to their destinations.
* AfterInstall, which occurs after files have been deployed.
* ApplicationStart, which happens after installation is complete
* ValidateService, which happens after the application has been started.

* Accordingly first we will install dependencies so create a file under scripts name as install_dependencies
* so accordingly {getting nginx,git,docker,dockercompose and setting up all required dependencies and docker containers.}
* __#!/bin/bash__
* __sudo apt update__
* __sudo apt-get Install nginx__
* __sudo apt install git__
* __sudo apt install curl__
* __curl -fsSL https://get.docker.com/ | sh__
* __sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
* __if [ ! -d "/usr/local/bin/docker-compose" ];then__
*    __sudo chmod +x /usr/local/bin/docker-compose__
* __fi
* __if [ ! -d "/opt/docker/rocket.chat/data/runtime/db" ];then__
*   __sudo mkdir -p /opt/docker/rocket.chat/data/runtime/db__
* __fi__
* __if [ ! -d "/opt/docker/rocket.chat/data/dump" ];then__
*   __sudo mkdir -p/opt/docker/rocket.chat/data/dump__
* __fi

* Now lets set up for after install:setup docker-compose file and setting up the domain . The code is uploaded into the github in the name of deafult1 and docker-compose1.yml.
* __#!/bin/bash__
* __sudo certbot certonly --standalone --email xxxxx@gmail.com -d mydomain.com__
* (
*    __cd /etc/nginx/sites-available/__
*    __cp default1 default__
*    __cd opt/docker/rocket.chat/__
*    __cp docker-compose1.yml docker-compose.yml__
* )
### Finally start server using
* __!/bin/bash__
* __service ngnix start__
* __if [ ! -d "/cd /opt/docker/rocket.chat" ];then__
*    __docker-compose up -d__ 
*    __docker run --name db -d mongo:4.0 mongod --smallfiles__
*    __docker run --name rocketchat -p 80:3000 --env ROOT_URL=http://mydomain --link db:db -d rocket.chat__
   
* __fi__
* with this setup +configuring IAM--->made finally to deploy on EC2 from codepipeline

### Deployment on aws Ec2instance using codedeploy:

* First go to the search toolbar and type IAM Role
* Here we have to create 2 differnet roles one for code deploy and another for EC2 instance
* Choose role which is present on the left side of the dashboard and select EC2
* Next select EC2 and choose  roles for cloning the github on your instance.
* Tag as name and value as EC2cosedeploy .these steps make it ready for EC2
* similarly create for codedeploy selecting aws access code deploy role and tag as name and value as codedeployrole.


### Setting up the Aws environment:

* First set up an Aws account on Amazon web services.
* After setting it up go to __“open Ec2 service ”__,click on Instances to the left side of your Ec2 dashboard and select an Ec2 instance.
* Now go to the search bar and search for __“Ubuntu Server 18.04 LTS”__ with 64 -bit (x86) Architecture.
* Select your instance(t2 -medium) and click __“Next: Configure Instance “__
* In this section scroll down and  see i am a role in the dropdown select it and give the following  as a text
* __yum -y update__
* __yum install -y ruby__ 
* __yum install -y aws-cli__ 
* __cd /home/ec2-user wget https://aws-codedeploy-us-east-2.s3.us-east-2.amazonaws.com/latest/install chmod +x ./install ./install auto__

* The above code installs the CodeDeploy agent on your instance as it is created.
* In the storage or choose your storage according to your usage.
* In this section set everything as  default.click __“Next “__
* This is Security Group add HTTP and HTTPS using their drop downs.click on __“Review and Launch”__
* This leads to creation of a key pair, now select  generate a new key pair and name the key pair as you like, keep the key pair secure since the key pair allows you to securely SSH into your instance. Finally __“Click on launch Instances”__.

### Configuring Elastic IP:
* Now click on  __view instances__  and you can see the instance running. 
* On your Ec2 dashboard you can see a section called Elastic iP click on that.
* Click __“Allocate new Address”__. Click __“Allocate”__.
* Copy the Ip address and save it. On the top find the __Actions button__, click on it and select __Associate address__.
* Now you can notice a  drop down Instance and private Ip select what it shows .(if you have multiple  then see which instances are running followed by their ip ).

### Configuring Route 53:
* Buy a domain either from aws or any other domain hosting company
* Go to create hosted zone on dashboard of route 53
* Create Record set 
* type : __Cname__, value: __Public Dns__ .


 
### Now it is time to Code deploy:

* Type Code Deploy on services toolbar
* Now you can see the code deploy dashboard in that select applications and create application.
* Name your application as rocketapp and platform to deploy as EC2 with this application is created


### Now deployment group has to be set 
* Now go through all dropdown and choose EC2 insatnce and tag name as your EC2 instance name, no need for load balancer this will create a deployment group.

### Last step create a codepipeline:
* choose the name of your pipline
* click on next ,source provider : choose git and follow steps and choose the repo which has your deployment file.
* Bild provider keeop it as default and in deployer part  choose code deploy. And this is enough to set your codepipeline

* finally start the deployment after a couple of min you can see the deployment success that means you have succesfully deployed rocketchat on your instance.






### TO clean up:
* Sign in to the AWS Management Console and open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
*    1. In the navigation pane, under Instances, choose Instances.
*    2. Select the box next to the Amazon EC2 instance you want to terminate. In the Actionsmenu, point to Instance State, and then choose Terminate.
*    3. When prompted, choose Yes, Terminate.
#### Default file
* server {
* listen 443 ssl;
* server_name mydomain.com;
* ssl_certificate /etc/letsencrypt/live/mydomain.com/fullchain.pem;
* ssl_certificate_key /etc/letsencrypt/live/mydomain.com/privkey.pem;
* ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
* ssl_prefer_server_ciphers on;
* ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
* root /usr/share/nginx/html;
* index index.html index.htm;
* #Make site accessible from http://localhost/
* server_name localhost;
* location / {
* proxy_pass http://localhost:3000/;
* proxy_http_version 1.1;
* proxy_set_header Upgrade $http_upgrade;
* proxy_set_header Connection "upgrade";
* proxy_set_header Host $http_host;
* proxy_set_header X-Real-IP $remote_addr;
* proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
* proxy_set_header X-Forwarded-Proto http;
* proxy_set_header X-Nginx-Proxy true;
* proxy_redirect off;
* }
* }
* server {
* listen 80;
* server_name mydomain.com;
* return 301 https://$host$request_uri;
* }

#### Docker1.yml
*  version: '2'

* services:
*   rocketchat:
*      image: rocket.chat:latest
*     command: >
*       bash -c
*         "for i in `seq 1 30`; do
*            node main.js &&
*            s=$$? && break || s=$$?;
*            echo \"Tried $$i times. Waiting 5 secs...\";
*            sleep 5;
*          done; (exit $$s)"
*    restart: unless-stopped
*      volumes:
*       - ./uploads:/app/uploads
*     environment:
*       - PORT=3000
*       - ROOT_URL=https://mydomain.com
*       - MONGO_URL=mongodb://mongo:27017/rocketchat
*       - MONGO_OPLOG_URL=mongodb://mongo:27017/local
*     depends_on:
*      - mongo
*     ports:
*       - 3000:3000

*   mongo:
*     image: mongo:4.0
*     restart: unless-stopped
*     command: mongod --smallfiles --oplogSize 128 --replSet rs0 --storageEngine=mmapv1
*     volumes:
*       - ./data/runtime/db:/data/db
*       - ./data/dump:/dump

*   #this container's job is just to run the command to initialize the replica set.
  
*   mongo-init-replica:
*     image: mongo:4.0
*     command: >
*       bash -c
*         "for i in `seq 1 30`; do
*           mongo mongo/rocketchat --eval \"
*             rs.initiate({
*               _id: 'rs0',
*               members: [ { _id: 0, host: 'localhost:27017' } ]})\" &&
*           s=$$? && break || s=$$?;
*           echo \"Tried $$i times. Waiting 5 secs...\";
*           sleep 5;
*         done; (exit $$s)"
*     depends_on:
*     - mongo


## Manual Method
### Setting up the Aws environment:

* First set up an Aws account on Amazon web services.
* After setting it up go to __“open Ec2 service ”__,click on Instances to the left side of your Ec2 dashboard and select an Ec2 instance.
* Now go to the search bar and search for __“Ubuntu Server 18.04 LTS”__ with 64 -bit (x86) Architecture.
* Select your instance(t2 -medium) and click __“Next: Configure Instance “__
* Keep it default in the storage or choose your storage according to your usage.
* In this section set everything as  default.click __“Next “__
* This is Security Group add HTTP and HTTPS using their drop downs.click on __“Review and Launch”__
* This leads to creation of a key pair, now select  generate a new key pair and name the key pair as you like, keep the key pair secure since the key pair allows you to securely SSH into your instance. Finally __“Click on launch Instances”__.

### Configuring Elastic IP:
* Now click on  __view instances__  and you can see the instance running. 
* On your Ec2 dashboard you can see a section called Elastic iP click on that.
* Click __“Allocate new Address”__. Click __“Allocate”__.
* Copy the Ip address and save it. On the top find the __Actions button__, click on it and select __Associate address__.
* Now you can notice a  drop down Instance and private Ip select what it shows .(if you have multiple  then see which instances are running followed by their ip ).

### Configuring Route 53:
* Buy a domain either from aws or any other domain hosting company
* Go to create hosted zone on dashboard of route 53
* Create Record set 
* type : __Cname__, value: __Public Dns__ .

### Connecting to your instance:

* Go to your Instance and  click connect, go to SSH client and follow the steps. 
* open your terminal(linux) or putty for windows.
* Now connect to your server using __chmod 400 “Your key”__ this command is executable only if your in the same directory of your key present.
* SSH the command, copy and paste it in your terminal  and say yes for connecting.
* Congratulations  you are  logged into your server. 




# Deployment Rocket chat:

### Getting an SSL Certificate:
* First type __sudo su__ for root user
* Installing certbot with the following commands:  __sudo apt update__ ,
   __sudo apt install certbot__
* To get your SSL certificate from “letsencrypt”  use: 
__sudo certbot certonly --standalone --email xxxxx@gmail.com -d mydomain.com__
* So,  etc/letsencrypt/archive creates 4 pem  files
   * Cert.pem
   * Fullchain.pem------(admins certificate file)
   * Chain.pem
   * Privkey.pem---------(certificate key file)


### Configure Nginx web server :

* Now __sudo apt-get Install nginx__
* Enable it by __sudo ufw enable__ and say yes to it
* Lets check the status by __sudo ufw status__ and see the __status : Active__
* __sudo ufw app__ list gives list of available applications
* Now if you want all then __sudo ufw allow ‘nginx full’__  ( for individual application change with ‘nginx name_app’)
* Type __sudo ufw status__ to see available applications along with where and action.
* Type __systemctl status nginx__ this function shows green light enabled and is active for both  webserver and reverse proxy. This statement conveys that the ip address of our ipv4 has started working.
* Check by typing ipv4 address on any browser on our computer and you can see it.
*  Backup the default config file using cd /etc/nginx/sites-available followed with sudo mv default default.reference
* Creating a new site configuration for Rocket.Chat 
* __sudo nano etc/nginx/sites-available/default__ which will open a blank file and paste the following given below
* server {
* listen 443 ssl;
* server_name mydomain.com;
* ssl_certificate /etc/letsencrypt/live/mydomain.com/fullchain.pem;
* ssl_certificate_key /etc/letsencrypt/live/mydomain.com/privkey.pem;
* ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
* ssl_prefer_server_ciphers on;
* ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
* root /usr/share/nginx/html;
* index index.html index.htm;
* #Make site accessible from http://localhost/
* server_name localhost;
* location / {
* proxy_pass http://localhost:3000/;
* proxy_http_version 1.1;
* proxy_set_header Upgrade $http_upgrade;
* proxy_set_header Connection "upgrade";
* proxy_set_header Host $http_host;
* proxy_set_header X-Real-IP $remote_addr;
* proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
* proxy_set_header X-Forwarded-Proto http;
* proxy_set_header X-Nginx-Proxy true;
* proxy_redirect off;
* }
* }
* server {
* listen 80;
* server_name mydomain.com;
* return 301 https://$host$request_uri;
* }

* Here mydomain.com is my domain name. Save this file
* Test the Nginx  sudo nginx -t where you will get a message : etc/nginx/nginx.conf syntax ok and test is successful

### Install GIT:
* __sudo apt install git__
* __git --version__
* __git clone https://github.com/RocketChat/Docker.Official.Image.git__

### Install Docker:
* __sudo apt-get update__
* __$ curl -fsSL https://get.docker.com/ | sh__



### Install Docker compose:
* __sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose__
* __sudo chmod +x /usr/local/bin/docker-compose__

### Setting up docker containers:

* __sudo mkdir -p /opt/docker/rocket.chat/data/runtime/db__
* __sudo mkdir -p /opt/docker/rocket.chat/data/dump__
* After these  commands create a docker-compose.yml 
* __sudo nano /opt/docker/rocket.chat/docker-compose.yml__
* Copy this and paste it below :
*  version: '2'

* services:
*   rocketchat:
*      image: rocket.chat:latest
*     command: >
*       bash -c
*         "for i in `seq 1 30`; do
*            node main.js &&
*            s=$$? && break || s=$$?;
*            echo \"Tried $$i times. Waiting 5 secs...\";
*            sleep 5;
*          done; (exit $$s)"
*    restart: unless-stopped
*      volumes:
*       - ./uploads:/app/uploads
*     environment:
*       - PORT=3000
*       - ROOT_URL=https://mydomain.com
*       - MONGO_URL=mongodb://mongo:27017/rocketchat
*       - MONGO_OPLOG_URL=mongodb://mongo:27017/local
*     depends_on:
*      - mongo
*     ports:
*       - 3000:3000

*   mongo:
*     image: mongo:4.0
*     restart: unless-stopped
*     command: mongod --smallfiles --oplogSize 128 --replSet rs0 --storageEngine=mmapv1
*     volumes:
*       - ./data/runtime/db:/data/db
*       - ./data/dump:/dump

*   #this container's job is just to run the command to initialize the replica set.
  
*   mongo-init-replica:
*     image: mongo:4.0
*     command: >
*       bash -c
*         "for i in `seq 1 30`; do
*           mongo mongo/rocketchat --eval \"
*             rs.initiate({
*               _id: 'rs0',
*               members: [ { _id: 0, host: 'localhost:27017' } ]})\" &&
*           s=$$? && break || s=$$?;
*           echo \"Tried $$i times. Waiting 5 secs...\";
*           sleep 5;
*         done; (exit $$s)"
*     depends_on:
*     - mongo

* After the above Start the container by 
   * __cd /opt/docker/rocket.chat__
   * __sudo docker-compose up -d__
* The above two will start rocket chat and mongo db 
 
### Individual Containers:
*First, start an instance of mongo:
 __docker run --name db -d mongo:4.0 mongod --smallfiles__
* Then start Rocket.Chat linked to this mongo instance:
 __docker run --name rocketchat --link db:db -d rocket.chat__

* This will start a Rocket.Chat instance listening on the default Meteor port of 3000 on the container.If you'd like  to access the instance directly at standard port on the host machine:
*  __docker run --name rocketchat -p 80:3000 --env ROOT_URL=http://mydomain --link db:db -d rocket.chat__
* Then, access it via http://localhost in a browser. Replace localhost in ROOT_URL with your own domain name if you are hosting at your own domain.
* Finally:
Login to your site at https://mydomain.com ---it is finally deployed on aws.







# How to remove the deployment and cleanup all resources created for it:

 ### Remove Docker(images,containers and volumes):
* __sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce__
* The above command will not remove images, containers, volumes, or user created configuration files on your host. If you wish to delete all images, containers, and volumes run the following commands:
* __sudo rm -rf /var/lib/docker /etc/docker__ 
*__sudo rm /etc/apparmor.d/docker sudo groupdel docker__
* __sudo rm -rf /var/run/docker.sock__
 
### Remove Git :
*__sudo apt-get remove --auto-remove__ (git all dependency packages too)
* To delete configuration and/or data files of git and it's dependencies execute:__$ sudo apt-get purge --auto-remove git__
 
# Remove Ngnix:
__sudo apt purge nginx__ 
This commands deletes all configurations .
 
# Remove Certbot:
__sudo certbot delete__
 
# Remove certbot files manually
* __sudo rm -rf /etc/letsencrypt/__
* __sudo rm -rf /var/lib/letsencrypt/__
* __sudo rm -rf /var/log/letsencrypt/__
 
# Make sure the repo is updated and autoremove
*__sudo apt update__
*__sudo apt upgrade__
*__sudo apt autoremove__
 
 
 
* These steps can clean up everything that was created.
 
 
 
## Additional methods:
 
# Remove Docker compose:
__sudo rm $(which docker-compose)__

# Remove all docker containers:
* Clean Docker and start from scratch, enter the command:
 __docker container stop $(docker container ls –aq) && docker system prune –af ––volumes__
 
 
 
 
 
 
 
 




 


