---
layout: "post"
title: "Creating a Security Testing Environment"
comments: true
date: "2019-06-19 07:49"
---

The goal of this exercise is to set up an intentionally vulnerable application called Damn Vulnerable Web Application (DVWA).

### Prerequisites

To set up this environment, you should have a basic understanding of networking, virtualization, client-server architecture, and a server-side programming language -- ideally PHP. You could possibly get by without this, but I'd recommend reassessing your priorities instead.  

You should also be comfortable with the Linux Command Line Interface (CLI).

### Disclaimer

I'm not responsible for anything you do with the information here, and the developers of DVWA are not responsible for how you use their application. 

> <span class="warning">**IMPORTANT**</span>: DVWA is a vulnerable web application. Do not upload it to your hosting provider's public html folder or any internet-facing server. 

Read the [DVWA Disclaimer](https://github.com/ethicalhack3r/DVWA#disclaimer) before continuing.

### Environment Details

The environment we're creating has two virtual machines. One acts as a server and serves DVWA. The other  acts as a client and accesses the DVWA application over a network as a typical user would. The network is internal and only allows traffic between virtual machines.

- No traffic to or from the internet
- No traffic to or from the host machine

The table below contains a list of software that I'll be working with. 

Software                   | Version
---------------------------|---------------------------------------------------------------------------------------------------------
Virtualization             | [Virtualbox 6.0.8 ](https://www.virtualbox.org/wiki/Downloads)
Server Operating System    | [Ubuntu Server 18.04.2 LTS](https://www.ubuntu.com/download/server)
Client Operating System    | [Kali Linux 2019.2 ](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/)
Web Server                 | [Apache Version: 2.4.29](https://httpd.apache.org/download.cgi)
Database Management System | [MySQL Version: 5.7.26](https://dev.mysql.com/downloads/)
Server-side Language       | [PHP Version: 7.2.19](https://www.php.net/downloads.php)
Vulnerable Web App         | [Damn Vulnerable Web Application v1.9](https://github.com/ethicalhack3r/DVWA)

You don't need to run the same software or versions I am if you're not interested in following along with any of my upcoming demos. Check the links in the table above to find the latest software releases.

I've not tested this guide on Windows or macOS, but the only difference should be the VirtualBox installation.

## Installing VirtualBox

VirtualBox is a free and open source virtualization software that allows you to run one or more guest machines inside a host machine. The purpose of using virtual machines, in this case, is to contain the exploitable application that you're installing to your machine.

Navigate to the [VirtualBox website](https://www.virtualbox.org/wiki/Linux_Downloads) and download the appropriate package for your host operating system. I'm assuming you can handle the installation yourself.

## Creating Virtual Machines

Download an operating system to use as the server. I chose [Ubuntu Server 18.04.2 LTS](https://www.ubuntu.com/download/server) because it's commonly used and Debian-based. You can use another Linux operating system, however, the server installation steps may vary from this guide as a result. 

> **Note**: If you want to use software such as [XAMPP](https://www.apachefriends.org/index.html) for your server and database, I recommend using [Lubuntu](https://lubuntu.net/) instead of Ubuntu Server. Lubuntu is lightweight compared to Ubuntu but still has a graphical environment, unlike Ubuntu Server.

After you have an operating system, follow the steps below to create a new virtual machine:

1. Open VirtualBox and Select **New**.
2. Give the VM a name; I'm using 'DVWA'.
3. Set the type to **Linux**.
4. Choose the appropriate version of your guest operating system -- **Ubuntu (64-bit)** in my case -- then click **Next**.
5. Select an amount of RAM to allocate to the guest machine. Ubuntu Server requires at least 512MB. The default 1024MB is fine.
6. Follow the remaining steps of the creation process. The defaults options are fine.
7. Click **Create** to complete the creation.
8. Select the new machine and click **Start**. You will be prompted to select a virtual optical disk to start the machine from.
9. Select the operating system you downloaded earlier, then click **Start**.
10. Follow the Ubuntu installation process. The default options are fine. Don't select any of the popular snaps in server environments.

Complete this process again for the client machine. I'm using [Kali Linux 2019.2 ](https://www.offensive-security.com/kali-linux-vm-vmware-virtualbox-image-download/), but you can use any operating system. 

### Optimizing the Client (Optional)

After you've created the client machine, right-click it within VirtualBox and click **settings**. You may want to increase the number of processors allocated to this machine under **System** > **Processor** > **Processor(s)** for improved performance. 

You may also want to increase the video memory, under **Display** > **Screen** > **Video Memory**.

## Managing the Server

Run the following commands in the server VM.

1. Update the operating system.

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo apt update && sudo apt upgrade`

2. Install the server and application dependencies.

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo apt-get -y install git apache2 mysql-server php php-mysqli php-gd libapache2-mod-php`

3. Move to **/var/www/html** and clone [DVWA](https://github.com/ethicalhack3r/DVWA) from GitHub using Git.

    {:style="overflow: auto; white-space: nowrap;"}
    `cd /var/www/html`

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo git clone https://github.com/ethicalhack3r/DVWA`

> **Troubleshooting**: If you can't connect to the internet, make sure your host machine is connected to the internet and your guest machines are set to the default networking option, NAT.

## Managing the Client

Update the client VM with the following command:

{:style="overflow: auto; white-space: nowrap;"}
`sudo apt update && sudo apt upgrade`

## VirtualBox Networking Modes

Virtualbox's default networking mode **Network Address Translation (NAT)**. In a NAT networking mode, a VM is able to connect to the internet through the VirtualBox networking engine, which maps traffic from the guest machine to the host machine. While the VM can access the host machine and internet, it is not accessible _from_ the internet or host machine. Using this setting would be sufficient if you want to both serve and test DVWA from the same virtual machine. 

To access the virtual machine from the host machine in a NAT networking mode, port forwarding would need to be enabled. This would allow bidirectional traffic to and from the VM. Configuring port forwarding would allow you to serve DVWA from a guest machine and test it from the host machine, however, it is less secure in that it allows traffic to the VM and relies on a firewall to filter that traffic.

### Choosing a Networking Mode

[The DVWA project README](https://github.com/ethicalhack3r/DVWA#warning) recommends running DVWA in a NAT networking mode. It doesn't specify whether this means serving and testing the application within the same VM or configuring port forwarding in the VM and testing the application from the host machine. I suspect the former is more secure, but they link an [installation video](https://github.com/ethicalhack3r/DVWA#installation-videos) which enables port forwarding.

As far as I know, DVWA does not require internet access to function. I also want to serve the application from one guest machine and test it from another guest machine. For these reasons, I'm going to deviate from the DVWA README's recommendation and run it in an **Internal Network**. In an internal network:

- One or more VMs can connect to each other 
- No traffic is permitted between any VM and the host machine or any VM and the internet, effectively isolating the VM(s).

> **NOTE**: [Click here to learn more about VirtualBox's networking modes.](https://www.virtualbox.org/manual/ch06.html#networkingmodes)

## Isolating the VMs within an Internal Networking

### Creating a DHCP Server

A Dynamic Host Configuration Protocol (DHCP) server assigns IP addresses to each device within the network and is required to connect to the server from the client.

To create a DHCP server:

1. Poweroff all guest machines.
2. Run the following command on the host machine to create a new DHCP server.

    {:style="overflow: auto; white-space: nowrap;"}
    `vboxmanage dhcpserver add -netname intnet --ip 10.10.10.1 --netmask 255.255.255.0 --lowerip 10.10.10.2 --upperip 10.10.10.10 --enable`

> **NOTE**: [Click here to learn more about the VBoxManage dhcpserver command.](https://www.virtualbox.org/manual/ch08.html#vboxmanage-dhcpserver)

### Adding the VMs to the Internal Network
Complete the following steps on both the client and server machines:

1. Open Virtualbox.
2. Right-click the machine and select **Settings**.
3. Move to the **Network** section and under the **Adapter 1** tab, set the **Attached to:** setting to **Internal Network**.
4. Set the network's name to **intnet**, if it's not already, then click **OK**.

> **NOTE**: The VMs will not be able to connect to the internet while set to Internal Network. If you need to download software or updates, set the VM to NAT networking mode temporarily. 

### Connecting to the Server from the Client

Start the server and client VMs and login to your accounts.

In the server:

1. Confirm that Apache is running:

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo systemctl status apache2`
  
2. If the Apache is inactive, start it: 

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo systemctl start apache2`

3. Find the IP address of the machine: 

    {:style="overflow: auto; white-space: nowrap;"}
    `ifconfig`. 

     The IP address is likely next to **Inet** and should look like **10.10.10.x** where **x** can be any number between 1 and 10 if you configure the DHCP server using the same parameters I did.

In the client:

1. Open a browser.
2. Go to the server's IP address by typing it into the address bar. 

If your internal network is successfully configured and Apache is running in the server VM, you should be greeted with the Apache welcoming page.

## Configuring the server

1. In the client, move to the DVWA page, **10.10.10.2/DVWA** for me. You should receive the following error message: 

    *DVWA System error - config file not found. Copy config/config.inc.php.dist to config/config.inc.php.*

2. Following the error message directions, copy the configuration file using the following command within the server:

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo cp /var/www/html/DVWA/config/config.inc.php.dist /var/www/html/DVWA/config/config.inc.php`

3. In the client, refresh the page. You should be redirected to **10.10.10.2/DVWA/setup.php**, which contains an installation checklist.

4. Click **Create / Reset Database**

    We haven't configured MySQL or DVWA so you should receive the following error message: 
    
    *Could not connect to the MySQL service. Please check the config file.*

### Creating a new MySQL User

We need to create a new user to use when connecting to the dvwa database. Run the following commands in the server VM:

1. Open MySQL.

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo mysql`

2. Create the DVWA database.

    {:style="overflow: auto; white-space: nowrap;"}
    mysql> `CREATE DATABASE dvwa;`

3. Create a new user called 'dvwa' with the password 'password123' -- you can use any valid username or password.

    {:style="overflow: auto; white-space: nowrap;"}
    mysql> `GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost' IDENTIFIED BY 'password123';`
    
4. Close mysql

    {:style="overflow: auto; white-space: nowrap;"}
    mysql> `exit`
    
### Setting the Database Credentials in DVWA

1. Open the DVWA configuration file in a text editor, by running the following command in the server VM:

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo vim /var/www/html/DVWA/config/config.inc.php`
    
2. Set the following database variables:

    <!-- 
    Variable               | Value         |
    -----------------------|---------------|--
    $_DVWA['db\_server']   | '127.0.0.1'   |
    $_DVWA['db\_database'] | 'dvwa'        |
    $_DVWA['db\_user']     | 'dvwa'        |
    $_DVWA['db\_password'] | 'password123' | 
    --> 
    
    <!-- ======= HTML for text-alignment consistency ======= -->
    <table>
      <thead>
        <tr>
          <th style="text-align: center;">Variable</th>
          <th style="text-align: center;">Value</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>$_DVWA[‘db_server’]</td>
          <td>‘127.0.0.1’</td>
        </tr>
        <tr>
          <td>$_DVWA[‘db_database’]</td>
          <td>‘dvwa’</td>
        </tr>
        <tr>
          <td>$_DVWA[‘db_user’]</td>
          <td>‘dvwa’</td>
        </tr>
        <tr>
          <td>$_DVWA[‘db_password’]</td>
          <td>‘password123’</td>
        </tr>
      </tbody>
    </table>
    <!-- ================== END Of TABLE ======================== -->

3. Save and close the file.
    
4. In the client VM, refresh the installation page and click **Create / Reset Database**. 

    > **NOTE**: Configuring the remaining checklist items on the setup page is covered in the next section of this guide. The database can be reset later by clicking this button again.

  If your database is set up correctly you will see a message at the bottom of the page stating '_Setup Successful!_' before being redirected to the login page.

## Configuring DVWA According to the Setup Check

The configurations below are required to test some of DVWA's vulnerabilities. Please check [The DVWA project README](https://github.com/ethicalhack3r/DVWA#warning) for any additional changes that may be suggested after the publication of this guide.

### PHP function allow_url_include

Enabling allow_url_include is required for file-inclusion testing. To turn this on, perform the following actions in the server VM:

1. Edit the php.ini file. The location of this file may vary depending on your operating system and version of PHP.

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo vim /etc/php/7.2/apache2/php.ini`

2. Replace **allow_url_include = off** with **allow_url_include = on** within the php.ini.

3. Save and close the file, then restart Apache.

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo systemctl restart apache2`

### reCAPTCHA key

reCAPTCHA keys must be added to DVWA's **config.inc.php** file to test for reCAPTCHA vulnerabilities.

[Click here to generate reCAPTCHA keys](https://www.google.com/recaptcha/admin/create).

**reCAPTCHA v2** is supposedly functional, according to [pull request #227](https://github.com/ethicalhack3r/DVWA/pull/227). I recommend checking the [DVWA issues page](https://github.com/ethicalhack3r/DVWA/issues) for any new information on **reCAPTCHA v3**. Otherwise, select **"I'm not a robot" Checkbox** under **reCAPTCHA v2**.

To add reCAPTCHA keys to DVWA's config file:

1. Edit the file within the server VM:

    `sudo vim /var/www/html/DVWA/config/config.inc.php`
    
2. Set **$_DVWA['recaptcha_public_key']** and **$_DVWA['recaptcha_private_key']** to the keys you generated using the link above.

3. Save and close the file.

### File Permissions

The Apache user must have write permission to a few directories to work with file uploads and the PHP-Intrusion Detection System (PHPIDS).

1. Find the apache2 user. 

    {:style="overflow: auto; white-space: nowrap;"}
    `ps aux | egrep '(apache|httpd)'`
    
    Mine is **www-data**.

2. Recursively set the **/var/www/** folder's user/group owner to the apache user.

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo chown -R www-data:www-data /var/www`

3. Check with **/var/www/** file permissions.

    {:style="overflow: auto; white-space: nowrap;"}
    `ls -la /var/www`
    
    It should look similar to **drwxr-xr-x**. This string is divided into four parts:
    
    - **d** - Indicates that this file is a directory.
    - **rwx** - Indicates the user, www-data in this case, has read, write, and execute permission.
    - **r-x** - Indicates the user group, also www-data, has write and execute permission.
    - **r-x** - Indicates all users have read and execute permissions.

4. If your www-data user does not have write permission (w), assign it with the following command:

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo chmod -R 755`

> **NOTE**: [Click here to learn more about Linux file permissions.](https://www.linux.com/learn/understanding-linux-file-permissions)

### .htaccess Configuration

DVWA's .htaccess file must be changed for SQL injection to work in PHP v5.2.6 or above.

1. Edit the .htaccess file.

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo vim /var/www/html/DVWA/.htaccess`

2. Replace:
    
    ```
    <IfModule mod_php5.c>
        php_flag magic_quotes_gpc off
        #php_flag allow_url_fopen on
        #php_flag allow_url_include on
    </IfModule>
    ```

    With:

    ```
    <IfModule mod_php5.c>
        magic_quotes_gpc = Off
        allow_url_fopen = On
        allow_url_include = On
    </IfModule>
    ```
    
3. Save and close the file.

### Recreate the Database

After making all of the necessary configurations to DVWA and the server, recreate the dvwa database.

1. In the client VM, open **10.10.10.2/DVWA/setup.php**.

2. Click **Create / Reset Database**.

## Using DVWA

Login to DVWA with the default admin credentials **admin** and **password**. The navigation bar on the left lists the vulnerability modules. The application's security setting is currently set to **impossible**, meaning that these vulnerabilities do not exist.

### Changing Security Settings

To change DVWA's difficulty settings:

1. Open **config.inc.php** in the server.

    {:style="overflow: auto; white-space: nowrap;"}
    `sudo vim /var/www/html/DVWA/config/config.inc.php`

2. Set **$_DVWA['default_security_level']** to 'low', 'medium', 'high', or 'impossible', depending on your needs.

3. **(OPTIONAL)** Set **$_DVWA['default_phpids_level']** to 'enable' if you want to practice testing an application that is monitored by an intrusion detection system.

4. **(OPTIONAL)** Set **$_DVWA['default_phpids_verbosel']** to 'true' if you want to be notified when your request(s) has been blocked by PHPIDS.

5. Save and close the file.

## Troubleshooting

Leave a comment below if you have any problems with this guide.

