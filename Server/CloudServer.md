---
layout: default
title: Cloud Server
---

# BUILDING A CLOUD SERVER

**Operating System:** Linux

## Prerequisites
Web Server already set on a Linux virtual machine (VM).

For this guide, we'll use **Nextcloud**.

Download it from https://nextcloud.com/install/

---

- Log in as superuser, unzip the folderand move it into the web server directory:
    ```ps
    mv nextcloud /var/www/html
    chmod -R 777 /var/www/html/nextcloud
    ```

- Open browser and log into phpMyAdmin as root. Add a new user:
    ```
    Username: nextcloud
    Password: <PASSWORD>
    Privileges: all
    ```
    and enable the option `create a database with the same name` 

- Through your browser, search `https://<PUBLIC_IP>/nextcloud` and register with these credentials:
    ```
    Username: <USERNAME>
    Password: <PASSWORD>
    DB User: nexcloud	 
    DB Name: nexcloud
    DB Pass: <PHP_MY_ADMIN_PASSWORD>
    ```

- You can also set Nextcloud web page as your server index page:
    ```ps
    echo "<?php
            header(‘Location: https://<your hostname>/nextcloud’)
    >?" > /var/www/html/index.php
    ```
