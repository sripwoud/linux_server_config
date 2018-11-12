# Linux Server Configuration
This page explains how to set up a secure linux web server using a Digital
Ocean droplet. This server will then be used to deploy my vinyls catalogue
previously developed.  
## 1. Create the Digital Ocean Droplet
1. On your local machine create RSA key pairs with the following command:  
```
$ ssh-keygen
```
2. [Create digital ocean droplet](https://www.digitalocean.com/docs/droplets/how-to/create/)  
I chose the following configuration: 1 GB Memory / 25 GB Disk / FRA1 - Ubuntu 18.04 x64.  
Provide the ssh `.pub` key you created at the previous step.
3. [Log in to your droplet with ssh](https://www.digitalocean.com/docs/droplets/how-to/connect-with-ssh/):
```  
$ ssh -i /path/to/private/key root@ip.ad.dr.ess
```

## 2. Initial server set up
As explained in this [tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04), there are a few basic initial configuration steps that should be taken over in order to secure the server:
1. Create a new user:  
```
$ adduser username
```
2. Grant sudoer rights:
```
$ usermod -aG sudo username
```
4. Update and upgrade (keeping local existing configuring files, if asked):
```
$ apt update && apt upgrade
```
5. Enable external access via ssh for the newly created user by copying the ssh public key:  
```
$ rsync --archive --chown=username:username ~/.ssh /home/username
```
6. Change ssh port from 22 to 2200:
```
$ nano /etc/ssh/sshd_config
```
Replace `#port 22` by `port 2200`.
7. Restart:
```
service ssh restart
```
8. ```exit```
9. Log in as the new via ssh:
```
ssh -i /path/to/private/sshkey username@ip.ad.re.ss -p 2200
```
10. Configure the firewall to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) and OpenSSH:
```
$ sudo ufw default deny incoming  # deny all incoming connections
$ sudo ufw default allow outgoing # allow all outgoing connections
$ sudo ufw allow OpenSSH
$ sudo ufw allow 2200/tcp         # open port 2200 for ssh connection
$ sudo ufw allow www              # open port 80 for HTTP requests
$ sudo ufw allow 123/udp
$ sudo ufw deny 22                # close this port as it was replaced by 2200
$ sudo ufw enable
$ sudo ufw status                 # should list all the rules you just set up
```  
11. Disable root login  
```
$ sudo nano /etc/ssh/sshd_config
```  
Replace `PermitRootLogin yes` by `PermitRootLogin no`.
```
$ service ssh restart  
$ ssh -i /path/to/private/sshkey root@ip.ad.dr.ess -p 2200
```
Should not work anymore.
12. Configure Timezone to Use UTC  
```
sudo dpkg-reconfigure tzdata
```  
Select `none of the above`.

## 3. Install Apache web server  
```
$ sudo apt update  
$ sudo apt install apache2
```  
Check your website at http://ip.ad.dr.ess.
You should see the default apache 'it works' page.
## 4. Install mod_wsgi
```
$ sudo apt install libapache2-mod-wsgi
```    
## 5. Install pip
The application I am deploying is written in python 2.7.
```
$ sudo apt install python-pip  
```
Check with `pip --version` that the installation was successful.  
2. [Install and configure git](https://git-scm.com/download/linux)  
```
$ sudo add-apt-repository ppa:git-core/ppa
$ sudo apt update
$ sudo apt-get install git
$ git config --global user.name "username"
$ git config --global user.email "username@mail.com"  
```

10. check: git config -l
11. install postgresql
https://www.postgresql.org/download/linux/ubuntu/   
nano /etc/apt/sources.list.d/pgdg.list  
add line: deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main  
import repository signing key:  
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update  
sudo apt install postgresql-10  
1. configure postgresql:
    - sudo su - postgres
    - psql
    - CREATE ROLE catalog WITH PASSWORD 'catalog';
    - ALTER ROLE catalog CREATEDB;  
    check with \du that it is been created  
    \q  
    exit  
1. create catalog user and add to sudoers  
sudo adduser catalog
sudo usermod -aG sudo catalog
1. create catalog db:  
su - catalog  
createdb catalog  
check with \l
1. create app directory:  
sudo mkdir /var/www/catalog && cd /var/ww/catalog/
1. clone repo:  
sudo git clone https://github.com/Gry0u/vinyls_catalogue.git catalog
1. chown -R grader:grader catalog/
1. cd catalog/catalog
1. mv views.py __init__.py
1. edit  
'# app.debug = True
   # app.run(host='0.0.0.0', port=5000)
    app.run()'  
1. edit database_setup.py:  
engine = create_engine('postgresql:///catalog:catalog@localhost/catalog',
                       connect_args={'check_same_thread': False})
1. add server to Google API authorized origins
1. update client_secrets.json by downloading it, copying content and creating
a new client_screts.json in catalog/catalog. check that the client_secret key
is in the json file, if necessary add it manually
1. update login.html if necessary
1. create python virtual env  
http://flask.pocoo.org/docs/0.12/installation/  
sudo apt install python-virtualenv  
in catalog/catalog/: sudo virtualenv venv  
sudo chown -R grader:grader venv/  
. venv/bin/activate  
sudo pip install --upgrade flask sqlalchemy httplib2 oauth2client requests  
sudo apt install libpq-dev  
sudo pip install psycopg2










[intial server set up digital ocean ink](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04?utm_source=local&utm_medium=Email_Internal&utm_campaign=Email_UbuntuDistroNginxWelcome&mkt_tok=eyJpIjoiTXpZNE5EaGpOR0ZoT1dFeiIsInQiOiIzc2ZXZVlHanE3Sk85ZWpmaEN3cUJSa21ZNlkwUGJJMVBZMkVkZHVmQTNZSzczUTdEYnpTYStTNWpaajg0MGJ3SUhHTlVSTlR3OGY5RzBhVlVBem5lRmp0WHJkdURHZmNuN0JHeTR3c2s0TjAxUlZcL043UDNXS3RIbndITTQwTFMifQ%3D%3D)

https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps