# Linux Server Configuration Project

## Overview

This repo will explain how to set up a Linux machine virtually and configure it to host a webpage. The webpage used here is from the previous project from the same coursework - [item-catalog](https://github.com/prkksh/item-catalog/) (This project is for Linux Server Configuration from Udacity Full Stack Developer Nanodegree).

Public IP: 3.211.93.109

## Configuration steps

### Lightsail Setup

* Go to AWS [Lightsail](https://lightsail.aws.amazon.com) and create a new account / sign in with your account. Click `Create instance` and choose `Linux/Unix`,`OS only` `Ubuntu 16.04LTS` (there are multiple other options to choose).

* Before Creating the instance make sure to save the SSH key. You can click on the `change SSH key pair` to download the key or upload your own key.
* Save the key under the name `lightsail_key.rsa`
* Click on Create and wait for the virtual machine to start.
* Run a command `ssh -i lightsail_key.rsa ubuntu@3.211.93.109` in your terminal.

### Update the packages

* Run the following to update all available packages
```
sudo apt-get update
sudo apt-get upgrade
```

### Change SSH port from 22 to 2200

* Edit `/etc/ssh/sshd_config` file by `sudo nano /etc/ssh/sshd_config`

* Find line `Port 22` and change it to `Port 2200`
* Save the change by `Control + X ` and exit from nano.
* Restart SSH with ` sudo service ssh restart`

### Set up Uncomplicated Fire Wall (UFW)

* Configure UFW to allow only incoming request from port2200(SSH), port80 (HTTP) and port123 (NTP).
```
sudo ufw status
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
```

* Go to AWS page and set up relevant ports from `networking` tab.
* Select the custom port and set it to 2200

### Create a new user called grader and give an access

* Run `sudo adduser grader` to create a new user called `grader`

* Create a new file in sudoer directory with `sudo nano /etc/sudoers.d/grader`
* Add `grader ALL=(ALL:ALL) ALL` in nano editor


### Create a SSH key pair

* Create a key pair with `ssh-keygen` locally in your machine.

* Copy the generated SSH to the virtual environment.
On your virtual machine:
```
$ su - grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ vim .ssh/authorized_keys
```
Copy the public key generated on your local machine to this file and save
```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```
* Reload SSH using `service ssh restart`
* Now you can use ssh to login with the new user you created
* To login as grader - `ssh -i [privateKeyFilename] grader@3.211.93.109`

### Disable root login

* Open `/etc/ssh/sshd_config` and find `PermitRootLogin` and change it to `no`.


### Set up local time zone

* Run `sudo dpkg-reconfigure tzdata` and choose UTC


### Install Apache application and wsgi module

* Run `sudo apt-get install apache2` to install apache

* Run `sudo apt-get install python-setuptools libapache2-mod-wsgi` to install mod-wsgi module

* Start the server `sudo service apache2 start`


### Install git

* Run `sudo apt-get install git`

### Install and configure PostgreSQL

* Install PostgreSQL `sudo apt-get install postgresql`

* Check if no remote connections are allowed `sudo vim /etc/postgresql/9.5/main/pg_hba.conf`
* Login as user "postgres" `sudo su - postgres`
* Get into postgreSQL shell `psql`
* Create a new database named catalog  and create a new user named catalog in postgreSQL shell

	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
* Set a password for user catalog

	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
* Give user "catalog" permission to "catalog" application database

	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
* Quit postgreSQL `postgres=# \q`

* Exit from user `postgres`


### Clone the project

* Run `cd /var/www` and `sudo mkdir FlaskApp`

* Change the owner to grader `sudo chown -R grader:grader FlaskApp`

* Run `sudo chmod FlaskApp` to give a permission to clone the project.

* Switch to the `FlaskApp` directory and clone the Catalog project.

* `cd FlaskApp` and `git clone https://github.com/prkksh/item-catalog.git FlaskApp`

* Modify filenames to deploy on AWS.

* Rename `project.py` to `__init__.py`
```
sudo mv project.py  __init__.py
```


### Install Flask framework

* Install `pip`, `sudo apt-get install python-pip`

* Install Flask `pip install Flask` and dependencies `pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2.`


### Configure Apache

* Create a config file `sudo nano /etc/apache2/sites-available/FlaskApp.conf`

* Paste the following code
```
<VirtualHost *:80>
    ServerName Public-IP-Address
    ServerAdmin admin@Public-IP-Address
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

* Enable the new virtual host `sudo a2ensite FlaskApp`


### Create the .wsgi File

* Create the .wsgi File under /var/www/FlaskApp:

	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi
	```
* Add the following lines of code to the flaskapp.wsgi file:

	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/")

	from FlaskApp import app as application
	application.secret_key = 'Add your secret key'
	```

* Change the engine inside Flask application.
  `engine = create_engine('postgresql://catalog:password@localhost/catalog')`

* Set up the DB with `python /var/www/catalog/FlaskApp/database_setup.py`

### Restart Apache

* Run `sudo service apache2 restart` and check `http://3.211.93.109/`

###### Errors & Fix:
When running apache service, you might get an error saying `Invalid connection option 'check_same_thread'`. This error can be fixed by removing the option `connect_args={'check_same_thread': False}` in the `__init__.py` file.


## References
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)

[Daniel Abrão - linux_server_configuration](https://github.com/jungleBadger/-nanodegree-linux-server)


This project was developed in a windows operating system.
