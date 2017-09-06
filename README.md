# Udacity-Linux-Server-Configuration
FSND : Udacity Project 7

A Linux virtual machine(Amazon Lightsail / Ubuntu) is configured for Item Catalog website. 

You can visit the web site at http://54.146.228.92/ 

## Instructions for SSH access to the instance
	- Download Private Key below and move it into the folder `~/.ssh` .
	- Open your terminal and type in
		```chmod 600 ~/.ssh/My_Key```
	- In your terminal, type in
		```ssh -i ~/.ssh/My_Key.rsa @54.146.228.92```
	5. Development Environment Information

Public IP Address

54.146.228.92
	
Private Key ( is not provided here. )

## Create a new user named grader and give root access
	 `sudo adduser grader`
	 `vim /etc/sudoers`
	 `touch /etc/sudoers.d/grader`
	 `vim /etc/sudoers.d/grader`
	 	Type -- `grader ALL=(ALL:ALL) ALL`
	 	save and quit

## Set ssh login using keys
	- generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
	- deploy public key on developement enviroment

		On you virtual machine:
		```
		 su - grader
		 mkdir .ssh
		 vim .ssh/authorized_keys
		```
		Copy the public key generated on your local machine to this file and save
		```
		 chmod 700 .ssh
		 chmod 644 .ssh/authorized_keys
		```
	
	- reload SSH using `service ssh restart`
	- now you can use ssh to login with the new user you created

		`ssh -i path/to/key grader@54.146.228.92`

## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## Change the SSH port from 22 to 2200
- Use `sudo vim /etc/ssh/sshd_config` and change Port 22 to 2200 , save & quit using `:wq`.
- Reload SSH using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable 
 
## Configure the local timezone to UTC
	- Configure the time zone `sudo dpkg-reconfigure tzdata`
	- Select your timezone 
	- Selct city

## Install and configure Apache to serve a Python mod_wsgi application
	`sudo apt-get install apache2`
	`sudo apt-get install python-setuptools libapache2-mod-wsgi`
	`sudo service apache2 restart`

## Install and configure PostgreSQL
	- Install PostgreSQL `sudo apt-get install postgresql`
	- Get into postgreSQL shell `psql -u postgres`
	- Create a new database and user named `catalog` in postgreSQL shell. Set password and give access privilages
		
		```
		postgres=# CREATE DATABASE catalog;
		postgres=# CREATE USER catalog;
		postgres=# ALTER ROLE catalog WITH PASSWORD 'catalog';
		postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
		```
	
## Install git
	`sudo apt-get install git`

## Clone and setup your Catalog App project.

	- Use `cd /var/www` to move to the /var/www directory 
	- Create the application directory `sudo mkdir catalog`
	- Move inside this directory using `cd catalog`
	- Clone the Catalog App to the virtual machine `git clone https://github.com/gokusayon/item_cataougue.git`
	- Rename the project's name `sudo mv ./item_cataougue ./catalog`
	- Rename `main.py` to `__init__.py` using `sudo mv main.py __init__.py`
	- Edit `database_set_up.py` and `entityManagerService.py` and change
	 		`engine = create_engine('mysql+pymysql://root:root@localhost/item_catalog')` to 
	 		`engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`

## Installing dependencies
	- Install pip `sudo apt-get install python-pip`
	- Use pip to install dependencies `sudo pip install -r requirements.txt`
	- Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
	- Create database schema `sudo python database_setup.py`

## Configure and Enable a New Virtual Host
	-`sudo nano /etc/apache2/sites-available/catalog.conf`

	-Add the following lines of code to the file to configure the virtual host. 
			```
		<VirtualHost *:80>
			ServerName 54.146.228.92
			ServerAdmin gokusayon@gmail.com
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
	-Enable using command `sudo a2ensite catalog`

## Create the .wsgi File
		```
		cd /var/www/catalog
		sudo nano catalog.wsgi 
		```
	- Add the following lines of code to the catalog.wsgi file:
		
		```
		#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0,"/var/www/catalog/")

		from catalog import app as application
		application.secret_key = 'Add your secret key'
		```

## Restart Apache

	-Restart Apache `sudo service apache2 restart `

## Author
**Vasu Sheoran**