# Linux-Server-Configuration
Udacity Fullstack Nanodegree Project 5
ssh -v grader@52.11.176.33 -p2200
password: grader

## Procedure


### (1) Set up Linux server as Instructed by Udacity
<a href="https://www.udacity.com/account#!/development_environment">Udacity Account</a>

### (2) Create a new user named grader
<a href="https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps">Source</a>

Create user grader
```
$ adduser grader
```

### (3) Give the grader the permission to sudo
<a href="https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps">Source</a>

Edit sudoers file to allow grader to sudo
```
$ visudo
```
Find the line
```
root ALL=(ALL:ALL) ALL
```
and add
```
grader ALL=(ALL:ALL) ALL
```
below it

### (4) Update all currently installed packages

Use these commands to update installed packages
```
$ sudo apt-get update
$ sudo apt-get upgrade
```

### (5) Change the SSH port from 22 to 2200
<a href="http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server">Source</a>

open config file
```
nano /etc/ssh/sshd_config
```
Change port from 22 to 200
Change PermitRootLogin to no
Change PasswordAuthentication to yes
append these lines to the file
```
UseDNS no
AllowUsers grader
```
#### restart ssh
<a href="https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server">Source</a>

```
$service ssh restart
```

#### Create an SSH key pair for login as grader

```
$ ssh-keygen
$ ssh-copy-id grader@YOUR-IP-ADDRESSS -p2200
$ ssh -v grader@YOUR-IP-ADDRESSS -p2200
```

#### Reopen config file and change PasswordAuthentication to no

```
$ sudo vim /etc/ssh/sshd_config
```

### (6) Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
Open required ports then enable firewall.

```
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw enable
```

### (7) Configure the local timezone to UTC
Open time zone configuration dialog.

```
$ sudo dpkg-reconfigure tzdata
```

Choose none of the above and then UTC.


### (8) Install and configure Apache to serve a Python mod_wsgi application
<a href="http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html">Source</a>

Install Apache2, if you visit your servers website you shuld see the default page.
Istall mod_wsgi and restart apache2

```
$ sudo apt-get install apache2
$ sudo apt-get install python-setuptools libapache2-mod-wsgi
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo a2enmod wsgi
$ sudo service apache2 restart
```
### (9) Install and configure PostgreSQL:
<a href="https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps">Source</a>
* Do not allow remote connections
remote connections should already be blocked, double check by opening the config file

```
$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf
```

* Create a new user named catalog that has limited permissions to your catalog application database.

#### create linux user catalog

```
$ sudo adduser catalog
```

#### Switch to postgres user and open postgresql

```
$ su -u postgres -i
$ psql
```
#### Prompt should now end with #. Create new user and give it permission to create databases. check that the user was created properly then make the databse and restrict access to it.

```
# CREATE USER catalog WITH PASSWORD 'DB-PASSWORD'; //This password will be used later
# ALTER USER catalog CREATEDB;
# \du
# CREATE DATABASE catalog WITH OWNER catalog;
# \c catalog
# REVOKE ALL ON SCHEMA public FROM public;
# GRANT ALL ON SCHEMA public TO catalog;
# \q
```


### (10) Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

#### Install git and protect .git

```
$ sudo apt-get install git
```

#### Go to the www dirrectory and create a directory catalog

```
$ cd /var/www/
$ mkdir catalog
$ cd catalog
```

#### Make .git file inaccessable

```
$ sudo nano .htaccess
```

add line `RedirectMatch 404 /\.git`

#### use git clone to bring in web, rename the resulting dirrectory catalog

```
$ git clone ADDRESS-FROM-GITHUB-PROJECT-CLONE-BAR
$ mkdir catalog
$ sudo mv -r CLONED-DIRRECTORY/* catalog
$ cd catalog
```

#### change application.py to __init__.py

```
$ sudo mv application.py __init__.py
```

#### Edit engine in __init__.py, database_setup.py and db_populate.py
use `$ sudo nano` one each of the mentioned files, and find the line.

```
engine = create_engine('sqlite:///itemscatalog.db')
```

replace in each case with

```
engine = create_engine(postgresql://catalog:DB-PASSWORD@localhost/catalog')
```

#### Modify __init__.py so Google+ login works.

```
sudo nano __init__.py
```

find any reference to client_secret.json and replace it with its full path name

```
/var/www/catalog/catalog/client_secret.json
```

find the line `app.debug = True` and delete it.

#### Install required packages and modules. dont use sudo inside virtualenv as it may cause packages to install incorrectly
<a href="http://stackoverflow.com/questions/5420789/how-to-install-psycopg2-with-pip-on-python">Source</a>

```
$ sudo apt-get install python-pip
$ sudo apt-get install libpq-dev python-dev
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ sudo chmod -R 777 venv
$ source venv/bin/activate
$ pip install Flask
$ pip install httplib2
$ pip install requests
$ pip install flask-seasurf
$ pip install dict2xml
$ pip install --upgrade oauth2client
$ pip install sqlalchemy
$ pip install http://pypi.python.org/packages/source/p/psycopg2/psycopg2-2.4.tar.gz#md5=24f4368e2cfdc1a2b03282ddda814160
$ deactivate
```
create your database
```
$ python database_setup.py
$ python db_populate.py
```

#### Create a Virtual Host
<a href="https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps">Source</a>
Use '$ sudo nano /etc/apache2/sites-available/catalog.conf' and paste in

```
<VirtualHost *:80>
      ServerName YOUR-IP-ADDRESS
      ServerAdmin admin@YOUR-IP-ADDRESS
      ServerAlias HOSTNAME
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi
      <Directory /var/www/catalog/catalog/>
          Order allow,deny
          Allow from all
      </Directory>
      Alias /static /var/www/catalog/catalog/static
      <Directory /var/www/catalog/catalog/static/>
          Order allow,deny
          Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
```

and replace IP addresses and HOSTNAME

#### Create .wsgi file

```
$ cd /var/www/catalog
$ sudo nano catalog.wsgi
```

paste in

```
#!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'super_secret_key'
```

#### Enable virtual host and restart apache

```
$ sudo a2ensite catalog
$ sudo service apache2 restart
```

#### go to your <a href="https://console.developers.google.com/project">Google Console</a> and add your HOSTNAME to your project's Authorized Javascript Origins and Authorized Redirect URIs.

## Extra Credit

#### Include cron scripts to automatically manage package updates
<a href="https://help.ubuntu.com/community/AutomaticSecurityUpdates">Source</a>
Install and enable unattended-upgrades package.

```
$ sudo apt-get install unattended-upgrades
$ sudo dpkg-reconfigure -plow unattended-upgrades
```

#### Configure Firewall to monitor for repeated unsuccessful login attempts and ban attackers
<a href="https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04">Source</a>

Install fail2ban and configure its settings

```
$ sudo apt-get install fail2ban
$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
$ sudo nano /etc/fail2ban/jail.local
```

add these parameters

```
set bantime  = 1800
destemail = YOUR-EMAIL
action = %(action_mwl)s
under [ssh] change port = 2220
```
restart fail2ban
```
$ sudo service fail2ban start
```

#### Install Monitor Application
```
$ sudo pip install Glances
```
