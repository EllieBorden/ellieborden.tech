---
layout: "post"
title: "Creating a Security Testing Environment"
comments: true
date: "2019-06-06 07:49"
---

The goal of this exercise is to set up an intentionally vulnerable application called Damn Vulnerable Web Application (DVWA), which I will use in upcoming security testing demos.

### Overview

This guide walks through the following operations:

- [Creating the Virtual Machines (VMs)](#creating-the-virtual-machines)
  - One machine acting as a server and the other as a client
- [Updating the VMs and downloading the server and DVWA](#updating-the-machines-and-downloading-the-server-and-dvwa)
- [Isolating the VMs within an internal network](#isolating-the-machines-within-an-internal-network)
  - VMs can interact with each other
  - VMs cannot interact with the host or access the internet
- [Configuring the server](#configuring-the-server)
- [Accessing the server from the client](#accessing-the-server-from-the-client)
- [Configuring and install DVWA](#configuring-and-installing-dvwa)

 
I've not tested these on Windows or macOS, but the only difference should be the VirtualBox installation.

This guide does not cover setting up shared folders or SSH. If you have any questions, post them in the comments section below.

### Disclaimer

I'm not responsible for anything you do with the information here, and the developers of DVWA are not responsible for how you use their application. 

> <span class="warning">**IMPORTANT**</span>: DVWA is a vulnerable web application. Do not upload it to your hosting provider's public html folder or any internet-facing server. 

Read the [DVWA Disclaimer](https://github.com/ethicalhack3r/DVWA#disclaimer) before continuing.

### Prerequisites

To set up this environment, you should have a basic understanding of networking, virtualization, client-server architecture, and a server-side programming language -- ideally PHP. You could possibly get by without this, but I'd recommend reaccessing your priorities instead.  

You should also be comfortable with the Linux Command Line Interface (CLI).

### Environment Details

The table below contains a list of software that I'll be working with. 

Software                   | Version
---------------------------|---------------------------------------------------------------------------------------------------------
Virtualization             | [Virtualbox 6.0.8 ](https://www.virtualbox.org/wiki/Downloads)
Server Operating System    | [Ubuntu Server 18.04.2 LTS](https://www.ubuntu.com/download/server)
Client Operating System    | [Kali Linux 2019.2 ](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/)
Web Server                 | [Apache Version: 2.4.29](https://httpd.apache.org/download.cgi)
Database Management System | [MySQL Verion: 5.7.26](https://dev.mysql.com/downloads/)
Server-side Language       | [PHP Version: 7.2.17](https://www.php.net/downloads.php)
Vulnerable Web App         | [Damn Vulnerable Web Application v1.9](https://github.com/ethicalhack3r/DVWA)

You don't need to run the same software I am if you're not interested in following along with my demos. Check the links in the table above to find the latest software releases.

### Installing VirtualBox

VirtualBox is a free and open source virtualization software that allows you to run one or more guest machines inside of a host machine. The purpose of using a virtual machine, in this case, is to contain the exploitable application that you're installing to your machine.

Navigate to the [VirtualBox website](https://www.virtualbox.org/wiki/Linux_Downloads) and download the appropriate package for your host operating system. I'm assuming you can handle the installation yourself.

### Creating The Virtual Machines

First, download an operating system to use as the server. I chose [Ubuntu Server 18.04.2 LTS](https://www.ubuntu.com/download/server) because it's commonly used and Debian-based. You can use another Linux operating system, however, the server installation steps may vary from this guide as a result. 

> **Note**: If you want to use software such as [XAMPP](https://www.apachefriends.org/index.html) for your server and database, I recommend using [Lubuntu](https://lubuntu.net/) instead of Ubuntu Server. Lubuntu is lightweight compared to Ubuntu but still has a graphical environment, unlike Ubuntu Server.

After you have an operating system, follow the steps below to create a new virtual machine:

1. Open VirtualBox and Select **New**.
2. Give the VM a name; I'm using 'DVWA'.
3. Set the type to **Linux**.
4. Choose the appropriate version of your guest operating system -- **Ubuntu (64-bit)** in my case -- then click **Next**.
5. Select an amount of RAM to allocate to the guest machine. Ubuntu Server requires a at least 512MB. The default 1024MB is fine.
6. Follow the remaining steps of creation process. The defaults options are fine.
7. Click **Create** to complete the creation.
8. Select the new machine and click **Start**. You will be prompted to select a virtual optical disk to start the machine from.
9. Select the operating system you downloaded earlier, then click **Start**.
10. Follow the Ubuntu installation process. The default options are fine. Don't select any of the popular snaps in server environments.

Complete this process again for the client machine. I'm using [Kali Linux 2019.2 ](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/), but you can use any operating system. 

#### Optimizing the Client (Optional)

After you've created the client machine, right click it within VirtualBox and click settings. You may want to increase the number of processors allocated to this machine under **System** > **Processor** > **Processor(s)** for improved performance. 

You may also want to increase the video memory, under **Display** > **Screen** > **Video Memory**.

### Updating the Machines and Downloading the Server and DVWA

Run the following commands in the corresponding VMs:

#### Server VM

Update the operating system.

{:style="overflow: auto; white-space: nowrap;"}
`sudo apt update && sudo apt upgrade`

Install the server and application dependencies.

{:style="overflow: auto; white-space: nowrap;"}
`sudo apt-get -y install git apache2 mysql-server php php-mysqli php-gd libapache2-mod-php`

Move to **/var/www/html** and clone [DVWA](https://github.com/ethicalhack3r/DVWA) from GitHub using Git.

{:style="overflow: auto; white-space: nowrap;"}
`cd /var/www/html`

{:style="overflow: auto; white-space: nowrap;"}
`sudo git clone https://github.com/ethicalhack3r/DVWA`

Run the following commands in the server VM:

#### Client VM

Update the server.

{:style="overflow: auto; white-space: nowrap;"}
`sudo apt update && sudo apt upgrade`

#### Troubleshooting

If you don't have internet, make sure your host machine is connected to the internet and your guest machines are set to the default networking option, NAT.

### Isolating the Machines Within an Internal Network

Run following command on the host machine. Give both networks intranet with the name 'intnet'

{:style="overflow: auto; white-space: nowrap;"}
`vboxmanage dhcpserver add -netname intnet --ip 10.10.10.1 --netmask 255.255.255.0 --lowerip 10.10.10.2 --upperip 10.10.10.10 --enable`

### Configuring the Server

sudo mysql
mysql> GRANT ALL PRIVILEGES ON *.* TO 'ellie'@'localhost' IDENTIFIED BY '[password]'
  mysql> SELECT User FROM mysql.user;
  
### Accessing the Server from the Client

### Configuring and Installing DVWA

Please check the [installation directions within the DVWA project-readme](https://github.com/ethicalhack3r/DVWA#installation).

Commands:
- cd /var/www/html
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