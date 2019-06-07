---
layout: "post"
title: "Installing a Security Testing Environment"
comments: true
date: "2019-06-06 07:49"
---

Go away. This is not finished.

### Agenda

The goal of this exercise is to set up a local security testing environment called Damn Vulnerable Web Application (DVWA), which I will in upcoming test-demos. 

The developers of DVWA recommend installing this application inside of a virtual machine that is set to NAT networking mode. That is what this walk-through covers, however, DVWA is also available as a [Docker container](https://github.com/ethicalhack3r/DVWA#download-and-install-as-a-docker-container).

><span class="warning">**IMPORTANT**</span>: DVWA is unsurprisingly a damn vulnerable web application. Do not upload it to your hosting provider's public html folder or any internet-facing server. 

I've not tested these installation steps on Windows or macOS, but the only difference should be the VirtualBox installation.

### Environment Details

**Virtualization Software**: [Virtualbox 6.0.8 ](https://www.virtualbox.org/wiki/Downloads)  
**Operating System**: [Ubuntu Server 18.04.2 LTS](https://www.ubuntu.com/download/server)  
**Server** [Apache Version: 2.4.29](https://httpd.apache.org/download.cgi)  
**Database Management System** [MySQL Verion: 5.7.26](https://dev.mysql.com/downloads/)  
**Server-side Language** [PHP Version: 7.2.17](https://www.php.net/downloads.php)  
**Web App**: [Damn Vulnerable Web Application v1.9](https://github.com/ethicalhack3r/DVWA)   

You don't necessarily need to run the same versions I am, but please be aware that variations in software may result in behavior that's inconsistent with my demos. Check the project links listed above to find the latest releases.

### Disclaimer

I'm not responsible for anything you do with the information here, and the developers of DVWA are not responsible for how you use their application. DVWA is a practice environment and should not be used maliciously. 

Read the [DVWA Disclaimer](https://github.com/ethicalhack3r/DVWA#disclaimer) before continuing.

### Prerequisites

Before installing DVWA, it would be helpful to have a basic understanding of virtualization, client-server architecture, and a server-side programming language -- ideally PHP. You could possibly get by without this, but I'd recommend reaccessing your priorities instead. 

You should also be comfortable with the Linux command line.

### Installing VirtualBox

VirtualBox is a free and open source virtualization software that allows you to run one or more guest machines inside of a host machine. The purpose of using a virtual machine, in this case, is to contain the exploitable application that you're installing to your machine.

Navigate to the [VirtualBox website](https://www.virtualbox.org/wiki/Linux_Downloads) and download the appropriate package for your host operating system. I'm assuming you can handle the installation yourself, but as a reminder for Linux users, a .deb package can be installed using the following command:

{:style="overflow: auto; white-space: nowrap;"}  
`sudo dpkg -i path/filename.deb`  

### Creating a Virtual Machine

First, download an operating system. I chose [Ubuntu Server 18.04.2 LTS](https://www.ubuntu.com/download/server) because it's commonly used and Debian-based. You can use another Linux operating system, however, the server installation steps may vary from this guide as a result. 

> If you want to use software such as [XAMPP](https://www.apachefriends.org/index.html) for your server and database, I recommend using [Lubuntu](https://lubuntu.net/) instead of Ubuntu Server. Lubuntu is lightweight compared to Ubuntu but still has a graphical environment, unlike Ubuntu Server.

After you have an operating system, follow the steps below to create a new virtual machine (VM):

1. Open VirtualBox and Select **New**.
2. Give the VM a name; I'm using 'DVWA'.
3. Set the type to **Linux**.
4. Choose the appropriate version of your guest operating system -- **Ubuntu (64-bit)** in my case -- then click **Next**.
5. Select an amount of RAM to allocate to the guest machine. The minimum requirement for Ubuntu Server is 512MB, so the default 1024MB is fine.

### Configuring the Virtual Machine

HTTP 80    -> 8080
MySQL 3306           -> 9306
SSH 22                      -> 2222

### Installing LAMP

Run the following commands within the virtual machine:
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql php-gd openssh git
sudo systemctl enable apache2
sudo systemctl enable mysql
sudo mysql_secure_installation
	Set the following settings to true:
		Password strength = Strong
		Remove anonymous users
		Disallow root login remotely
		Remove test database and access to it
		Reload privilege tables now
sudo systemctl enable ssh
mysql -V
apache2 -V
php -v
sudo systemctl restart apache2 - Result: active (running)
sudo systemctl status apache2 - Result: active (running)
sudo systemctl status mysql - Result: active (running)
sudo systemctl status ssh - Result: active (running)

NOTE: 
	If your servers are not active, try running sudo systemctl start 'service'
	SSH was installed uring the server installation, but I included the commands here incase it was not installed on your system.

<!--	
NOTE TO SELF: 
	SSH is unnecessary. Remove?
-->

### Configuring MySQL

sudo mysql
mysql> GRANT ALL PRIVILEGES ON *.* TO 'ellie'@'localhost' IDENTIFIED BY '[password]'
  mysql> SELECT User FROM mysql.user;
  
### Configuring Apache

### Installing DVWA

Please check the [installation directions within the DVWA project-readme](https://github.com/ethicalhack3r/DVWA#installation).

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