# Linux-Server-Configuration

Script that takes a baseline Linux Server and automates the configuration that secures the system from a number of attack vectors, serves a Postgresql database server and an Apache mod-wsgi server.

## App Information

The server ip address is: 52.11.84.161, SSH port is: 2200
The URL is: http://52.11.84.161/ or ec2-52-11-84-161.us-west-2.compute.amazonaws.com

## Tasks

In order to complete the project, the following task needed to be done.

1. Launch your Virtual Machine with your Udacity account
2. Follow the instructions provided to SSH into your server
3. Create a new user named grader
4. Give the grader the permission to sudo
5. Update all currently installed packages
6. Change the SSH port from 22 to 2200
7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
8. Configure the local timezone to UTC
9. Install and configure Apache to serve a Python mod_wsgi application
10. Install and configure PostgreSQL:
   	- Do not allow remote connections
11. Create a new user named catalog that has limited permissions to your catalog application database
	Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

## Task 1: Launch Virtual Machine

## Task 2: Instructions for SSH access

1. Download Private Key below
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
	```mv ~/Downloads/udacity_key.rsa ~/.ssh/```
3. Open your terminal and type in
	```chmod 600 ~/.ssh/udacity_key.rsa```
4. In your terminal, type in
	```ssh -i ~/.ssh/udacity_key.rsa root@52.11.84.161```

## Task 3: Create a new user named grader

`sudo adduser grader`

## Task 4: Give the grader the permission to sudo

`sudo usermod -a -G sudo grader`

## Task 5: Update all currently installed packages

`sudo apt-get update`
`sudo apt-get upgrade`

## Task 6: Change the SSH port from 22 to 2200
1. Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`

## Task 7: Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	`sudo ufw allow 2200/tcp`
	`sudo ufw allow 80/tcp`
	`sudo ufw allow 123/udp`
	`sudo ufw enable` 
 
## Task 8: Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`

## Task 9: Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Task 10: Install and configure PostgreSQL

1. Install PostgreSQL:`sudo apt-get install postgresql postgresql-contrib`

2. Configured not allow remote connections`sudo vi /etc/postgresql/9.3/main/pg_hba.conf`.  

3. I created a new user named catalog with limited permissions to catalog application database:

`su - postgres`

`psql`

'postgres=# CREATE DATABASE catalog;'

`postgres=# CREATE USER catalog;`

`postgres=# ALTER ROLE catalog WITH PASSWORD 'passwordgoeshere';`

`postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

## Task 11: Install git, clone and setup your Catalog App project.
1. `apt-get -y update && apt-get -y upgrade`
2. Install ```sudo apt-get install git apt-get -y install python-flask python-sqlalchemy && apt-get -y install python-psycopg2 && apt-get -y install python-pip && apt-get -y install python2.7-dev && apt-get -y install libjpeg-dev && apt-get -y install zlib1g-dev && pip install pillow && pip install oauth2client && pip install dicttoxml && pip install flask-seasurf```
3. `git clone [repository source]`
4. `cd /var/www' to move into directory`
5. `sudo mkdir FlaskApp` to make new folder
6. moved cloned repository in FlaskApp folder

## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file to configure the virtual host. 
	
	```
	<VirtualHost *:80>
                    ServerName [SERVER IP]
                    ServerAdmin [admin@mywebsite.com]
                    WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
                    ServerAlias [HOSTNAME]
                    <Directory /var/www/FlaskApp/FlaskApp/>
                        Order allow,deny
                        Allow from all
                    </Directory>
                    Alias /static /var/www/FlaskApp/FlaskApp/static
                    <Directory /var/www/FlaskApp/FlaskApp/static/>
                        Order allow,deny
                        Allow from all
                    </Directory>
                    ErrorLog ${APACHE_LOG_DIR}/error.log
                    LogLevel info
                    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/FlaskApp: 
`nano /var/www/FlaskApp/flaskapp.wsgi` 

2. Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/FlaskApp/FlaskApp/")

	from application import app as application
	application.secret_key = 'Add your secret key'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

# Additional Functions

## Set ssh login using keys
1. generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. deploy public key on developement enviroment

	On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ nano .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. reload SSH using `service ssh restart`

## Configure the firewall to monitor for repeated unsuccessful attempts, appropriately ban attackers and provide automated security feedback.

1. Change bantime to 1800 sec

`word="bantime[[:space:]]*=[[:space:]][[:digit:]]*"`
`rep="bantime = 1800"`
`sed -i "s/${word}/${rep}/" /etc/fail2ban/jail.local`

2. Change the email to my email

`word="destemail[[:space:]]*=[[:space:]]root@localhost"`
`rep="destemail = myemail@example.com"`
`sed -i "s/${word}/${rep}/" /etc/fail2ban/jail.local`

3. Change the ssh port in jail.local
`sed -i "/\[ssh\]/{N`
`N`
`N`
`s/\(\[ssh\]\n.*\n.*\)\n.*/\1\nport=2200/}" /etc/fail2ban/jail.local`
`sed -i "s/\(action_)/(action_mwl)/" /etc/fail2ban/jail.local`
`apt-get install -y sendmail`
`pip install Glances`

## Install unattended-packages and configure to auto-update
`apt-get install unattended-upgrades`
`dpkg-reconfigure -plow unattended-upgrades`
`apt-get install ntp`

## References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://github.com/robertavram/Linux-Server-Configuration/blob/master/readme.md