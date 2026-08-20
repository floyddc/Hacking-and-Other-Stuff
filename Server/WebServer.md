---
layout: default
title: Web Server
---

# BUILDING A WEB SERVER

**Operating Systems:** Windows & Linux

## Prerequisites
To build a home web server on Windows, you'll first need to install a Linux distribution inside a virtual machine (VM).

For this guide, we'll use **Debian 11**.

Download it from https://www.debian.org/distrib/

- [Create the VM](#1-create-the-vm)
- [Install Debian](#2-install-debian)
- [Verify Apache](#3-verify-apache)
- [Become Root](#4-become-root)
- [Configure SSL](#5-configure-ssl)
- [Install PHP](#6-install-php)
- [Install MySQL](#7-install-mysql)
- [Install phpMyAdmin](#8-install-phpmyadmin)
- [Install FTP Server](#9-install-ftp-server)
- [SSH Server](#10-ssh-access)
- [Configure a static private IP](#11-configure-a-static-private-ip)
- [Configure Port Forwarding](#12-configure-port-forwarding)
- [Configure Dynamic DNS](#13-configure-dynamic-dns)
- [Install the No-IP Client](#14-install-the-no-ip-client)
- [Update Apache redirect](#15-update-apache-redirect)

---

## 1. Create the VM
- Open **VirtualBox** and click **New**.
- Preliminary configuration:
   - **Name**: choose any name.
   - **Type.**: Linux.
   - **Version**: Debian (64-bit).
   - **Memory**: possibly 4GB but 2GB is sufficient.
   - **Virtual Disk**: **do not create one**.
- Click **Create**.
- VM configuration:
  - **Storage**: go to `Settings → Storage` and attach the downloaded Debian ISO as the IDE Controller.
  - **Network**: go to `Settings → Network` and select `Bridged Adapter`, so the VM uses the same network as your PC.
  - **Serial Ports**: enable `Settings → Serial Ports` by checking the corresponding box.
  - **USB**: enable `Settings → USB` and activate the USB Controller.

---

## 2. Install Debian
- Start the VM.
- If prompted, select the Debian ISO.
- Choose `Guided installation` and follow the installer.
- When asked which software to install, select:
    - GNOME Desktop Environment.
    - SSH Server.
    - Web Server.

---

## 3. Verify Apache
- Open a terminal and check your private IP: `ip addr show`.
- Copy your private IP address and open it in a web browser (`http://<PRIVATE_IP>`).

If Apache is working correctly, the default Apache page should appear.

---

## 4. Become Root
To perform administrative tasks: `su -` and enter your root password.

---

## 5. Configure SSL
- Create the SSL directory: `mkdir /etc/apache2/ssl`.
- Generate a self-signed certificate:
    ```
    openssl req -new -x509 -days 365 -nodes \
    -out /etc/apache2/ssl/apache.pem \
    -keyout /etc/apache2/ssl/apache.pem
    ```
- Fill in the requested information.
- Enable the required Apache modules:
    ```
    a2enmod ssl
    a2enmod rewrite
    ```
- Enable the SSL virtual host: `a2ensite default-ssl.conf`
- Open the default site configuration: `mousepad /etc/apache2/sites-available/000-default.conf`
- Add: `Redirect permanent / https://<PRIVATE_IP>/`.
- Save the file and restart Apache: `service apache2 restart`.
- Open the website again using HTTPS.

> Browsers will warn that the certificate is not trusted because it is self-signed. This is expected.

---

## 6. Install PHP
- Update repositories: `apt update`.
- Install PHP: `apt install php php-common`.
- Verify the installed version: `php -v`. 
- Install the Apache PHP module: `apt install libapache2-mod-php`.
- Enable PHP: `sudo a2enmod php<VERSION>`.

---

## 7. Install MySQL
Download the MySQL APT repository package from [mysql.com](https://dev.mysql.com/downloads/repo/apt)
- Install the downloaded package: `dpkg -i <PKG_NAME>`.
- Choose **OK** when prompted.
- Install MySQL Server: `apt install mysql-server`.
- Set a root password.
- Check the service status: `systemctl status mysql`.
- Verify the version: `mysqladmin -u root -p version`.

---

## 8. Install phpMyAdmin
- Install phpMyAdmin: `apt install phpmyadmin`.
- During installation:
    - Select **apache2**
    - Choose **YES** for dbconfig-common.
    - Set the administrator password.
- Access phpMyAdmin from your browser: [http://localhost/phpmyadmin](http://localhost/phpmyadmin) or `http://<PRIVATE_IP>/phpmyadmin`.
- Log in as:
    - **Username:** root.
    - **Password:** your MySQL root password.
- Create a new user with full privileges using: `Host: localhost`

---

## 9. Install FTP Server
- Install VSFTPD: `apt install vsftpd`.
- Open the configuration file: `mousepad /etc/vsftpd.conf`
- Modify:
    - Uncomment: `write_enable=YES`.
    - Add at the end: `local_root=/var/www/html`.
- Give full permissions: `chmod -R 777 /var/www/html`.
- Restart the service: `service vsftpd restart`.

---

### FileZilla configuration
- Download FileZilla from [filezilla-project.org](https://filezilla-project.org/download.php?platform=win64).
- Connection settings:

    | Field | Value |
    |--------|-------|
    | Host | Server private/public IP |
    | Username | Server username |
    | Password | Server password |
    | Port | 22 |

---

## 10. SSH access
- Download PuTTY from [putty.org](https://www.putty.org/).
- Connect using the same credentials used for FileZilla.

---

## 11. Configure a static private IP
- Open your router's administration page.
- Go to: `Local Network`.
- Assign a static IP address to the Debian machine.

---

### Debian network configuration
- Backup the current configuration: `cp /etc/network/interfaces /etc/network/interfaces.old`.
- Edit: `mousepad /etc/network/interfaces`.
- Add:
    ```
    allow-hotplug enp0s3

    iface enp0s3 inet static
    address <PRIVATE_IP>
    netmask <NETMASK>
    gateway <ROUTER_IP>
    ```
- Restart networking: `/etc/init.d/networking restart`.
- Bring up the interface: `ifup enp0s3`.
- Test the connection: `ping <PRIVATE_IP>` and then stop it with Ctrl+C.
- Stop the test with:
- Verify the configuration: `ip addr show`. You should see:
    ```
    valid_lft forever
    preferred_lft forever
    ```

---

## 12. Configure Port Forwarding
- Open your router's administration page.
- Navigate to: `WAN Services`.
- Add the following forwarding rules:

    | Service | Protocol | Port | Destination |
    |----------|----------|------|-------------|
    | Apache | TCP/UDP | 80 | Private IP |
    | Apache SSL | TCP/UDP | 443 | Private IP |
    | FTP | TCP/UDP | 21 | Private IP |
    | SSH | TCP/UDP | 22 | Private IP |

---

## 13. Configure Dynamic DNS
Your public IP is usually dynamic. To make your server reachable from anywhere, use a Dynamic DNS service such as [noip.com](https://www.noip.com).
- Create an account and:
    - Create a hostname.
    - Choose a domain.
    - Associate it with your current public IP (check it from [myip.com](https://www.myip.com)).
- Configure your router: `WAN Services → DNS & DynDNS`.
- Enable:
    - IPv4.
    - HTTPS.
- Insert your No-IP credentials.

---

## 14. Install the No-IP Client
- Install the required packages: `apt install build-essential manpages-dev`.
- Verify GCC: `gcc -v`.
- Download the client: 
    ```
    cd /usr/local/src
    wget http://www.no-ip.com/client/linux/noip-duc-linux.tar.gz
    ```
- Extract it: `tar xzf noip-duc-linux.tar.gz`.
- Enter the directory: `cd noip-2.1.9-1`.
- Compile: `make`.
- Install: `make install`.
- Configure the client: `/usr/local/bin/noip2 -C`.
- Start it: `/usr/local/bin/noip2`.

---

## 15. Update Apache redirect
- Edit: `mousepad /etc/apache2/sites-available/000-default.conf`.
- Replace: `Redirect permanent / https://<PUBLIC_IP>/` with: `Redirect permanent / https://<HOSTNAME>/`.
- Restart Apache: `service apache2 restart`.
- Test your website from another device using your hostname.