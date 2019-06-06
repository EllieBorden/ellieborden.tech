---
layout: "post"
title: "Installing LAMP and DVWA in VirtualBox"
comments: true
date: "2019-06-06 07:49"
---

Go away. This is not finished.

**TODO** 

- Follow requriements for DVWA starting at folder permissions
  https://github.com/ethicalhack3r/DVWA

- Look into Apache configurations
  https://www.lynda.com/Linux-tutorials/Configuring-Apache-HTTP-Server/587676/631539-4.html

### Virtual Machine

Software: Virtualbox 6.0  
Operating System: Ubuntu Server 18.04.2  

Ports:
- HTTP 80 -> 8080
- MySQL 3306 -> 9306
- SSH 22 -> 2222

### Server

Name: sqli-test  
Web App: DVWA   
Apache Version: 2.4.29  
MySQL Verion: 5.7.26  
PHP Version: 7.2.17  

### Installing LAMP

Command Log

*	sudo apt update && sudo apt upgrade -y
* sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql php-gd openssh git
*	sudo systemctl enable apache2
*	sudo systemctl enable mysql
*	sudo mysql_secure_installation
* Set the following settings to true:
  *	Password strength = Strong
  *	Remove anonymous users
  *	Disallow root login remotely
  *	Remove test database and access to it
  * Reload privlilege tables now
*	sudo systemctl enable ssh
*	mysql -V
*	apache2 -V
*	php -v
*	sudo systemctl restart apache2
  * Result: active (running)
*	sudo systemctl status apache2
	* Result: active (running)
*	sudo systemctl status mysql
	* Result: active (running)
*	sudo systemctl status ssh
	* Result: active (running)

NOTE: 
If your servers are not active trying running sudo systemctl start 'service'
SSH was installed uring the server installation, but I included the commands here incase it was not installed on your system.
	
NOTE TO SELF: 
SSH is unnecessary my purpose. Remove?


### Configuring MySQL

sudo mysql

mysql> GRANT ALL PRIVILEGES ON *.* TO 'ellie'@'localhost' IDENTIFIED BY '[password]'

mysql> SELECT User FROM mysql.user;


### Installing DVWA
Commands:
- cd /var/www/html
- sudo git clone https://github.com/ethicalhack3r/DVWA
- cd DVWA/config
- sudo cp config.inc.php.dist ./config.inc.php
- sudo vim config.inc.php
		   - Configure mysql server and credentials.:
	- db_user = 'ellie'
	- db_password = '[password]'
	- default_security_level = 'low'
	- default_phpids_level = 'disabled'
	- default_phpids_verbose = 'false'

- On your host machine, navigate to localhost:8080/DVWA/setup.php.
- Click 'Create / Reset Database'
	- If successful, you will be redirected to login.php.
	- Else cannot connect to database.

DVWA Folder Permission Requirements
- ./hackable/uploads/ - Needs to be writable by the web service (for File Upload).
- ./external/phpids/0.6/lib/IDS/tmp/phpids_log.txt - Needs to be writable by the web service (if you wish to use PHPIDS).