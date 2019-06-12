---
layout: "post"
title: "Creating a Security Testing Environment"
comments: true
date: "2019-06-06 07:49"
excerpt_separator: <!--more-->
---

The goal of this exercise is to set up an intentionally vulnerable application called Damn Vulnerable Web Application (DVWA) and walks through the following:

### Prerequisites

To set up this environment, you should have a basic understanding of networking, virtualization, client-server architecture, and a server-side programming language -- ideally PHP. You could possibly get by without this, but I'd recommend reaccessing your priorities instead.  

You should also be comfortable with the Linux Command Line Interface (CLI).

### Disclaimer

I'm not responsible for anything you do with the information here, and the developers of DVWA are not responsible for how you use their application. 

> <span class="warning">**IMPORTANT**</span>: DVWA is a vulnerable web application. Do not upload it to your hosting provider's public html folder or any internet-facing server. 

Read the [DVWA Disclaimer](https://github.com/ethicalhack3r/DVWA#disclaimer) before continuing.


### Environment Details

This environment has two virtual machines. One acts as a server and serves DVWA. The other  acts as a client and access the DVWA application over a network as a typical user would. The network is internal and only allows traffic between virtual machines.

- No traffic to or from the internet
- No traffic to or from the host machine

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

You don't need to run the same software or versions I am if you're not interested in following along with my demos. Check the links in the table above to find the latest software releases.

I've not tested these on Windows or macOS, but the only difference should be the VirtualBox installation.

## Installing VirtualBox

VirtualBox is a free and open source virtualization software that allows you to run one or more guest machines inside of a host machine. The purpose of using a virtual machine, in this case, is to contain the exploitable application that you're installing to your machine.

Navigate to the [VirtualBox website](https://www.virtualbox.org/wiki/Linux_Downloads) and download the appropriate package for your host operating system. I'm assuming you can handle the installation yourself.

## Creating Virtual Machines

Download an operating system to use as the server. I chose [Ubuntu Server 18.04.2 LTS](https://www.ubuntu.com/download/server) because it's commonly used and Debian-based. You can use another Linux operating system, however, the server installation steps may vary from this guide as a result. 

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

### Optimizing the Client (Optional)

After you've created the client machine, right click it within VirtualBox and click settings. You may want to increase the number of processors allocated to this machine under **System** > **Processor** > **Processor(s)** for improved performance. 

You may also want to increase the video memory, under **Display** > **Screen** > **Video Memory**.

## Managing the Server

Run the following commands in the corresponding VMs:

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

### Troubleshooting

If you don't have internet, make sure your host machine is connected to the internet and your guest machines are set to the default networking option, NAT.

## Managing the Client

Update the client with the following command:

{:style="overflow: auto; white-space: nowrap;"}
`sudo apt update && sudo apt upgrade`

## VirtualBox Networking Modes

Virtualbox's default networking mode **Network Address Translation (NAT)**. In a NAT networking mode, a VM is able to connect to the internet through the VirtualBox networking engine, which maps traffic from the guest machine to the host machine. While the VM can access the host machine and internet, it is not accessible _from_ the internet or host machine. Using this setting would be sufficent if you want to both serve and test DVWA from the same virtual machine. 

To access the virtual machine from the host machine in a NAT networking mode, port forwarding would need to be enabled. This would allow bidirectional traffic to and from the VM. Configuring port forwarding would allow you to serve DVWA from a guest machine and test it from the host machine, however, it is less secure in that it allows traffic to the VM and relies on a firewall to filter that traffic.

### Choosing a Networking Mode

[The DVWA project README](https://github.com/ethicalhack3r/DVWA#warning) recommends running DVWA in a NAT networking mode. It doesn't specify whether this means serving and testing the application within the same VM or configuring port forwarding in the VM and testing the application from the host machine. I suspect the former is more secure, but they link an [installation video](https://github.com/ethicalhack3r/DVWA#installation-videos) which enables port forwarding.

As far as I know, DVWA does not require internet access to function. I also want to serve the application from one guest machine and test it from another guest machine. For these reasons, I'm going to deviate from the DVWA README's recommendation and run it in an **Internal Network**. In an internal network:

- One or more VMs can connect to each other 
- No traffic is permitted between any VM and host or VM and internet, effectively isolating the VM(s).

> **NOTE**: [Click here to learn more about VirtualBox's networking modes.](https://www.virtualbox.org/manual/ch06.html#networkingmodes)

## Isolating the VMs within an Internal Networking



### Creating a DHCP Server

A Dynamic Host Configuration Protocl (DHCP) server assigns 
[dhcpserver](https://www.virtualbox.org/manual/ch08.html#vboxmanage-dhcpserver) IP addresses to each device within the network and is required to connect to the server from the client.

To create a DHCP server:

1. Poweroff all guest machines.
2. Run following command on the host machine to create a new DHCP server, which will assign IP address to guest machines within the internal network.

{:style="overflow: auto; white-space: nowrap;"}
`vboxmanage dhcpserver add -netname intnet --ip 10.10.10.1 --netmask 255.255.255.0 --lowerip 10.10.10.2 --upperip 10.10.10.10 --enable`

### Adding the VMs to the Internal Network
Complete the following steps for both the client and the server machines:

1. Open Virtualbox
2. Right click the machine and select **Settings**.
3. Move to the **Network** section and under the **Adapter 1** tab, set the **Attached to:** setting to **Internal Network**.
4. Set the network's name to **intnet**, if it is not already, then click **OK**.

> **NOTE**: The VMs will not be able to connect to the internet while set to Internal Network. If you need to download software or updates, set the VM to NAT networking mode temporarily. 

## Configuring the Server

sudo mysql
mysql> GRANT ALL PRIVILEGES ON *.* TO 'ellie'@'localhost' IDENTIFIED BY '[password]'
  mysql> SELECT User FROM mysql.user;
  
## Configuring and Installing DVWA

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