# Linux Server Configuration

### About the project
> A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers

* IP Address: [18.196.108.112](http://18.196.108.112/)
* SSH Port: 2200
* URL using DNS: [ec2-18-196-108-112.eu-central-1.compute.amazonaws.com](http://ec2-18-196-108-112.eu-central-1.compute.amazonaws.com/)


### Steps Followed to Configure the server

#### 1. Update all packages
```
sudo apt-get update
sudo apt-get upgrade
```
Enable automatic security updates
```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

#### 2. Change timezone to UTC and Fix language issues 
```
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8
```

#### 3. Create a new user grader and Give him `sudo` access
```
sudo adduser grader    pass: 112233
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
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable
```

#### 7. Install Apache2 and mod-wsgi for python and Git
```
sudo apt-get install apache2 libapache2-mod-wsgi git
```

#### 8. Install and configure PostgreSQL
```
sudo apt-get install libpq-dev python-dev
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
git clone https://github.com/AlyZahran/item_catalogue catalog
cd catalog
git checkout production
nano catalog.wsgi
```
Then add the following in `catalog.wsgi` file
```python
#!/usr/bin/python
import sys
sys.stdout = sys.stderr

sys.path.insert(0,"/var/www/catalog")

from app import app as application
```
Setup virtual environment and Install app dependencies 
```
sudo apt-get install python-pip
sudo -H pip install virtualenv
pip install -r requirements.txt
```
Edit Authorized JavaScript origins


#### 10. Configure apache server
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```
Then add the following content:
```
<VirtualHost *:80>
  ServerName 18.196.108.112
  ServerAlias ec2-18-196-108-112.eu-central-1.compute.amazonaws.com
  ServerAdmin alizahran@zezo.com
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

```


#### 11. Restart Apache Server
```
sudo service apache2 restart
```

### Resources
* [Stack Overflow](https://stackoverflow.com/)
* [Ask Ubuntu](https://askubuntu.com/)
* [Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
* [Set Up Apache Virtual Hosts on Ubuntu ](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
* [Fix locale issue](https://askubuntu.com/questions/162391/how-do-i-fix-my-locale-issue)
* [Flask mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
* [Apache Server Configuration Files](https://httpd.apache.org/docs/current/configuring.html)