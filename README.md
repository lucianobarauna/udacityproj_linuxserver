# Linux Server Configuration
A Udacity Fullstack Web Developer nanodegree project.

A baseline installation of a Linux server and prepare it to host my [web applications](https://github.com/lucianobarauna/udacityproj_item_catalogy).


# Amazon Lightsail
The Linux server instance choosed was Amazon Lightsail, with the following specifications:

- Name: udaprojLinux
- Linux: Ubuntu 18.04 LTS
- IP: 34.231.195.103
- SSH Port: 2200
- Reviewer user: grader
- Url: http://34.231.195.103/


# Summary
- [Create a server](#create-a-server)
    - [Create an aws lightsail instance](#create-an-aws-lightsail-instance)
    - [Configuring ports](#configuring-ports)
    - [Creating static ip](#creating-static-ip)
    - [Generating SSH for server connection (in local machine)](#generating-ssh-for-server-connection-in-local-machine)
    - [Create a new user (in aws console)](#create-a-new-user-in-aws-console)

- [Configure Security Server](#configure-security-server)
    - [Remove root user access](#remove-root-user-access)
    - [Setting up ssh port 2200](#setting-up-ssh-port-2200)
    - [Restart SSH service to active security changes above](#restart-ssh-service-to-active-security-changes-above)
    - [Now you can enter using ssh as grader](#now-you-can-enter-using-ssh-as-grader)
    - [Configure FIREWALL](#configure-firewall)
    - [Configure UTC TIME_ZONE](#configure-firewall)

- [Deploy catalog application on server](#deploy-catalog-application-on-server)
    - [Setup Apache to serve a Python mod_wsgi application](#setup-apache-to-serve-a-python-mod_wsgi-application)
    - [Setup PostgreSQL](#setup-postgresql)
    - [Install git to clone Catalog app project](#install-git-to-clone-catalog-app-project)
    - [Configure the Catalog app](#configure-the-catalog-app)
    - [Logging](#logging)

- [References and inspiration](#references-and-inspiration)
    - [GitHub Repositories](#github-repositories)
    - [Materials](#materials)

# Create a server

### Create an aws lightsail instance
- Enter [https://lightsail.aws.amazon.com/ls/webapp/home/instances](https://lightsail.aws.amazon.com/ls/webapp/home/instances)
- Choose: [Create Instance] -> [Linux] -> [Ubuntu 16.04] -> [Create Instance]
- Wait "Pending" status change

### Configuring ports
In the networks tab you need to configure the following ports:
- Custom - TCP - 2200

### Creating static ip
To create a static ip we need to use a DNS and the service [XPI.io](http://xip.io/) offers it to use for example in the application OAuth. This is a public service offered free of charge by Basecamp. An example would be: 54.84.49.254.xip.io


### Generating SSH for server connection (in local machine)
Set-up SSH keys for user grader

```
$ ssh-keygen grader-key
# Enter file name to save the key: grader
# Let's keep passphrase empty

$ cp grader-key ~/.ssh
#to save the private key in ~/.ssh on local machine
```

### Create a new user (in aws console)

1. Create a new user named grader
```
$ sudo adduser grader
#to create the user

$ sudo chage -d 0 grader
#to make grader change its pw at first login

$ sudo usermod -aG sudo grader
#to grant sudo to grader, add it to Sudoers
```

2. Adding ssh key to the created user
```
$ su - grader
#login with grader

$ mkdir .ssh

$ sudo touch .ssh/authorized_keys

$ sudo nano .ssh/authorized_keys
```
Copy the public key generated on your local machine to this file.

CTRL+O (save), ENTER (confirm), CTRL+X (exit nano)

3. Permissioning ssh files. Again, in aws console:
```
$ sudo chown -R grader.grader /home/grader/.ssh

$ sudo chmod 700 /home/grader/.ssh

$ sudo chmod 600 /home/grader/.ssh/authorized_keys

$ ls -als .ssh/
```

# Configure Security Server

### Remove root user access
Logged as grader:
```
$ sudo nano /etc/ssh/sshd_config
```

Change the following lines:
1. FROM: `PermitRootLogin without-password` TO: `PermitRootLogin no`
2. FROM: `PasswordAuthentication yes` TO: `PasswordAuthentication no`
3. ADD: `DenyUsers root`

`CTRL+O (save), ENTER (confirm), CTRL+X (exit nano)`


### Setting up ssh port 2200

Now, let's restart ssh and change its port to 2200:
```
$ sudo service ssh restart

$ sudo nano /etc/ssh/sshd_config
```
Change Port to 2200

`CTRL+O (save), ENTER (confirm), CTRL+X (exit nano)`

### Restart SSH service to active security changes above
```
$ sudo service ssh restart
```

### Now you can enter using ssh as grader:
The ssh key was placed in the "Notes to Reviewer" field body
```
ssh -i grader-key grader@34.192.184.204 -p 2200
```

### Configure FIREWALL
Logged as grader:

```
$ sudo ufw default deny incoming
#to block all incoming connections on all ports by default

$ sudo ufw allow 2200/tcp
#to allow incoming connection for SSH on port 2200

$ sudo ufw allow 80/tcp
#to allow incoming connection for port 80

$ sudo ufw allow 123/udp
#to allow incoming connection for port 80

$ sudo ufw enable
#to enable the firewall, use

$ sudo ufw status
#to make sure everything is fine
```

### Configure UTC TIME_ZONE
Logged as grader:
```
$ sudo timedatectl set-timezone UTC
```

# Deploy catalog application on server

Logged as grader:

### Setup Apache to serve a Python mod_wsgi application
```
$ sudo apt-get install apache2
#to install appache

$ sudo apt-get install python-setuptools libapache2-mod-wsgi
#to install Install mod_wsgi

$ sudo service apache2 restart
#to restart apache
```

### Setup PostgreSQL
```
$ sudo apt-get install postgresql
#to install postgresql

$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf
#to check if no remote connections are allowed

$ sudo su - postgres
#to login as postgres
```

- as postgree get into postgreSQL shell with ```psql``` in there do:

1. Create a new database named (in my case **restaurantmenulist**) and a user named **catalog**
```
postgres=# CREATE DATABASE restaurantmenulist;
```

2. Create a new user named catalog
```
postgres=# CREATE USER catalog;
```

3. Set a password for user catalog
```
postgres=# ALTER ROLE catalog WITH PASSWORD 'myNewPassword';
```
_I've also paste the myNewPassword on the "Notes to Reviewer" field._

4. Give user "catalog" permission to "catalog" application database
```
postgres=# GRANT ALL PRIVILEGES ON DATABASE restaurantmenulist TO catalog;
```

5. Quit postgreSQL
```
postgres=# \q
```

Exit from user "postgres"
```
exit
```

### Install git to clone Catalog app project.

```shell
$ sudo apt-get install git

$ cd /var/www

$ sudo mkdir ItemCatalog

$ cd ItemCatalog/

$ sudo git init
# Initiate an empty git repository in the current folder

$ sudo git remote add origin https://github.com/lucianobarauna/udacityproj_item_catalogy
# Add my Item Catalog App Repo as a remote repository

$ sudo git remote -v
# Check if the remote repository was added successfully

$ sudo git pull origin master
# Pull the remote repository

$ ls
# Check if the files were downloaded successfully

$ cd catalog/
```

### Configure the Catalog app

1. Rename ```project.py``` to```__init__.py```
``` shell
$ sudo mv project.py __init__.py
```
2. Edit database_setup.py, lotsofmenus.py, and the now renamed __init.py__, and change all occurrences of 'sqlite:///restaurantmenulist.db' to 'postgresql://catalog:password@localhost/restaurantmenulist', editing the files with: sudo nano 'FILE-NAME'

```python
# engine = create_engine('sqlite:///restaurantmenulist.db')
#to
create_engine('postgresql://catalog:grader@localhost/restaurantmenulist')
```
at

```shell
$ sudo nano __init__.py
$ sudo nano db_config.py
```
3. Install pip

```shell
$ sudo apt-get install python-pip
$ sudo pip install -r requirements.txt
```
- Then, use it to install all dependencies listed on requirements.txt
```
$ sudo pip install -r requirements.txt

```

5. Create database schema
```
$ sudo python database_setup.py
```

6. To fill the database
```
$ sudo python lotsofmenus.py
```

6. Configure Apache and Enable a New Virtual Host
Create FlaskApp.conf to edit:
```
$ sudo nano /etc/apache2/sites-available/ItemCatalog.conf
```
Add the following lines of code to the file to configure the virtual host.
```
<VirtualHost *:80>
        ServerName 34.231.195.103
        ServerAdmin baraunaluciano@gmail.com
        ServerAlias 34.192.184.204.xip.io
	WSGIScriptAlias / /var/www/ItemCatalog/itemcatalog.wsgi
	<Directory /var/www/ItemCatalog>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/ItemCatalog/static
	<Directory /var/www/ItemCatalog/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable the virtual host with the following command:
```
$ sudo a2ensite ItemCatalog
```

Activate the new configuration
```
$ sudo service apache2 reload
```

7. Create and config the *.wsgi file

To create AND edit the desired file:
```
$ sudo nano /var/www/ItemCatalog/itemcatalog.wsgi
```

add the following code to flaskapp.wsgi
```
#!/usr/bin/python
import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/ItemCatalog")

from __init__ import app as application
application.secret_key = 'super_secret_key'
```

# Logging
If something breaks or go wrong, you can access a log for this application with the following command:

```
sudo tail -f /var/log/apache2/error.log
```

# References and inspiration

### GitHub Repositories

[@mguidoti](https://github.com/mguidoti/FSND-p8-linux_server_configuration)

[@andrevst](https://github.com/andrevst/fsnd-p6-linux-server-configuration)

[@leandrocl2005](https://github.com/leandrocl2005/aws_lightsail_config_for_flask_python3)

[@jungleBadger](https://github.com/jungleBadger/-nanodegree-linux-server)

[@bcko](https://github.com/bcko/Ud-FS-LinuxServerConfig-LightSail)

[@rccmodena](https://github.com/rccmodena/linux_server_configuration)


### Materials
- [https://www.jetbrains.com/help/pycharm/managing-dependencies.html](https://www.jetbrains.com/help/pycharm/managing-dependencies.html)
- [https://pip.readthedocs.io/en/1.1/requirements.html](https://pip.readthedocs.io/en/1.1/requirements.html)
