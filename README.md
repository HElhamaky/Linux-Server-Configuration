# Linux Server Configuration Project

### About the project
> This is the last project toward Udacity Full Stack Web Developer Nanodegree. In this project, the [Item Catalog Application](https://github.com/HElhamaky/Item-Catalog-Project) from project 3 will be hosted by a Ubuntu Linux server on an Amazon Lightsail instance. A series of instructions will be presented below. You can visit http://15.188.77.140/ for the website deployed.

* IP Address: 15.188.77.140
* SSH Port: 2200

### Server Configuration Steps

#### 1. Update all packages
1. Run `sudo apt-get update` to update packages
2. Run `sudo apt-get upgrade` to install newest versions of packages
3. Set for future updates: `sudo apt-get dist-upgrade`
4. Enable automatic security updates running
`sudo apt-get install unattended-upgrades` 
 and `sudo dpkg-reconfigure --priority=low unattended-upgrades`

#### 2. Change timezone to UTC and Fix language issues 
```
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8
```

#### 3. Create a new user grader and Give him `sudo` access
```
sudo adduser grader
sudo nano /etc/sudoers.d/grader 
```
Then add the following text `grader ALL=(ALL) ALL`

#### 4. Setup SSH keys for grader
* On local machine 
`ssh-keygen`
Then choose the path for storing public and private keys
* On remote machine home as user grader
```
sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys 
sudo chmod 700 .ssh
sudo chmod 600 .ssh/authorized_keys 
nano .ssh/authorized_keys 
```
Then paste the contents of the public key created on the local machine

#### 5. Change the SSH port from 22 to 2200 | Enforce key-based authentication | Disable login for root user
```
sudo nano /etc/ssh/sshd_config
```
Then change the following:
* Find the Port line and edit it to 2200.
* Find the PasswordAuthentication line and edit it to no.
* Find the PermitRootLogin line and edit it to no.
* Save the file and run `sudo service ssh restart`

#### 6. Configure the Uncomplicated Firewall (UFW)

1. Check firewall status: `$ sudo ufw status`
2. Set default firewall to deny all incomings: `$ sudo ufw default deny incoming`
3. Set default firewall to allow all outgoings: `$ sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 to allow SSH: `$ sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 to allow www: `$ sudo ufw allow www`
6. Allow incoming UDP packets on port 123 to allow NTP: `$ sudo ufw allow ntp`
7. Close port 22: `$ sudo ufw deny 22`
8. Enable firewall: `$ sudo ufw enable`
9. Check out current firewall status: `$ sudo ufw status`
10. Update the firewall configuration on Amazon Lightsail website under **Networking**. Delete default SSH port 22 and add **port 80, 123, 2200**
11. Open up a new terminal and you can now ssh in via the new port 2200: `$ ssh -i <PRIVATE-KEY> grader@15.188.77.140 -p 2200`


#### 7. Install Apache2 and mod-wsgi for python3 and Git
```
sudo apt-get install apache2 libapache2-mod-wsgi-py3 git
```
#### 8. Install and configure PostgreSQL
```
sudo apt-get install libpq-dev python3-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
```
Then
```
CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```
**Note:** In your catalog project you should change database engine to
```
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```

#### 9. Clone the Catalog app from GitHub and Configure it
```
cd /var/www/
sudo mkdir catalog
sudo chown grader:grader catalog
git clone <your_repo_url> catalog
cd catalog
git checkout production # If you have a diffrent branch!
nano catalog.wsgi
```
Then add the following in `catalog.wsgi` file
```python
#!/usr/bin/python3
import sys
sys.stdout = sys.stderr

# Add this if you'll create a virtual environment, So you need to activate it
# -------
activate_this = '/var/www/catalog/env/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))
# -------

sys.path.insert(0,"/var/www/catalog")

from app import app as application
```
Setup virtual environment and Install app dependencies 
```
sudo apt-get install python3-pip
sudo -H pip3 install virtualenv
virtualenv env
source env/bin/activate
pip3 install -r requirements.txt
```
- If you don't have `requirements.txt` file, you can use
```
pip3 install flask packaging oauth2client redis passlib flask-httpauth
pip3 install sqlalchemy flask-sqlalchemy psycopg2 bleach requests
```

Edit Authorized JavaScript origins

#### 10. Configure apache server
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```
Then add the following content:
```
# serve Item catalog app
<VirtualHost *:80>
  ServerName <IP_Address or Domain>
  ServerAlias <DNS>
  ServerAdmin <Email>
  DocumentRoot /var/www/catalog
  WSGIDaemonProcess catalog user=grader group=grader
  WSGIScriptAlias / /var/www/catalog/catalog.wsgi

  <Directory /var/www/catalog>
    WSGIProcessGroup catalog
    WSGIApplicationGroup %{GLOBAL}
    Require all granted
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

# Serve another project on the server with different port (Extra Step)
LISTEN 8000
<VirtualHost *:8000>
  ServerName <IP_Address or Domain>
  ServerAlias <DNS>
  ServerAdmin <Email>
  DocumentRoot /var/www/map

 <Directory /var/www/map/.git>
    Require all denied
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```


#### 11. Reload & Restart Apache Server
```
sudo service apache2 reload
sudo service apache2 restart
```


### Citation and Attribution
* [Discuss Linux Server Configuration Project](https://youtu.be/v9VvJvTyuH0)
* [Linux-Server-Configuration](https://github.com/AliMahmoud7/linux-server-configuration) by @AliMahmoud7
* [Flask mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
* [Apache Server Configuration Files](https://httpd.apache.org/docs/current/configuring.html)
* [Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Set Up Apache Virtual Hosts on Ubuntu ](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
* [mod_wsgi documentation](https://modwsgi.readthedocs.io/en/develop/)
* [Automatic Security Updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates#Using_the_.22unattended-upgrades.22_package)

