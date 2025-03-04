# TechnicSolder
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
![Latest Stable Version](https://img.shields.io/badge/dynamic/json.svg?label=Latest%20Stable%20Version&url=https%3A%2F%2Fraw.githubusercontent.com%2FTheGameSpider%2FTechnicSolder%2Fmaster%2Fapi%2Fversion.json&query=version&colorB=brightgreen)
![Latest Dev Version](https://img.shields.io/badge/dynamic/json.svg?label=Latest%20Dev%20Version&url=https%3A%2F%2Fraw.githubusercontent.com%2FTheGameSpider%2FTechnicSolder%2FDev%2Fapi%2Fversion.json&query=version&colorB=orange)

>TechnicSolder is an API that sits between a modpack repository and the Technic Launcher. It allows you to easily manage multiple modpacks in one single location.

>Using Solder also means your packs will download each mod individually. This means the launcher can check MD5's against each version of a mod and if it hasn't changed, use the cached version of the mod instead. What does this mean? Small incremental updates to your modpack doesn't mean redownloading the whole thing every time!

>Solder also interfaces with the Technic Platform using an API key you can generate through your account there. When Solder has this key it can directly interact with your Platform account. When creating new modpacks you will be able to import any packs you have registered in your Solder install. It will also create detailed mod lists on your Platform page! (assuming you have the respective data filled out in Solder) Neat huh?

-- Technic

TechnicSolder was originaly developed by Technic using the Laravel Framework. However, the application is difficult to install and use. Technic Solder - Solder.cf by TheGameSpider runs on pure PHP with zip and MySQL extensions and it's very easy to use. To install, you just need to install zip extension, setup MySQL database and download Solder to your server (No composer needed). And the usage is even easier! Just Drag n' Drop your mods.

# Detailed Installation
> ***Note: If you already have a working web server with mysql and zip extensions and enabled rewrite mod, you can [skip to step 6.](https://github.com/TheGameSpider/TechnicSolder#cloning-technicsolder-repository)***

**1. Install Ubuntu Server 20.04 (https://releases.ubuntu.com/20.04/)** <br />
**2. Login to Ubuntu with credentials you set.** <br />
**3. Become root**
Root is basically the "god account" that controls everything on the system.
You should never, _EVER_ use root to do simple tasks, unless you want your computer to be destroyed.
```bash
sudo su -
```
## Web server installation and configuration
**4. Install Prerequisites**<br />
This command installs what's known as a LAMP Stack, which includes Apache2, MariaDB, and PHP. Very useful!
```bash
apt update
apt -y install apache2 php libapache2-mod-php mariadb-server php-mysql php-dev zlib1g-dev libzip5 libzip-dev php-zip
```
The above command can take a while to complete. Once done, restart apache.<br />

```bash
service apache2 restart
```

We're now going to test that Apache and PHP are working together. Open up a blank file:
```
nano /var/www/html/index.php
```
and put the following text, inside:
```php
<?php
phpinfo();
```
When you are finished, save and close the file. (``Ctrl-X, y, Enter``)<br />
Now we can test whether our web server can correctly display content generated by a PHP script. To try this out, we just have to visit this page in our web browser. You'll need your server's public IP address. If you haven't already, and need to, remember to port forward port 80 (TCP).
```bash
curl http://icanhazip.com
```
The above command shows your WAN address and will not work as your address if you dont' have your ports forwarded right to that server/machine. If your doing proxying like me use your local server ip to see if PHP is working. Example http://192.168.3.10/index.php

Open in your web browser: `http://your_server_IP_address` <br />
This page basically gives you information about your PHP Compiler. It is useful for debugging and to ensure that your settings are being applied correctly.<br />
<br />
You probably want to remove this file after this test because it could actually give information about your server to unauthorized users. To do this, you can type
```bash
rm /var/www/html/index.php
```
**5. Enable RewriteEngine**<br />
```bash
a2enmod rewrite
nano /etc/apache2/sites-enabled/000-default.conf
```
Add this before `</VirtualHost>` close tag:
```
    DirectoryIndex index.php index.html
    <Directory /var/www/html>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Require all granted
    </Directory>
```
Save and close the file
## Cloning TechnicSolder repository
**6. Clone TechnicSolder repository** 
```bash
cd /var/www/
git clone https://github.com/82280zx/TechnicSolder.git html
```
Installation is complete. Now you need to confige TechnicSolder before using it.
# If you are using nginx:
*there is an example for nginx configuration"*
```nginx
	location / {
        try_files   $uri $uri/ /index.php?$query_string;
        }

	location /api/ {
        try_files   $uri $uri/ /api/index.php?$query_string;
        }

    location ~* \.php$ {
            fastcgi_pass                    unix:/run/php/php7.2-fpm.sock;
            fastcgi_index                   index.php;
            fastcgi_split_path_info         ^(.+\.php)(.*)$;
            include                         fcgi.conf;
            fastcgi_param PATH_INFO         $fastcgi_path_info;
            fastcgi_param SCRIPT_FILENAME   $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
            deny all;
    }

    location ~ .*/\. {
            return 403;
    }

    error_page 403 /403.html;
    
    location ~* \.(?:ico|css|js|jpe?g|JPG|png|svg|woff)$ {
            expires 365d;
	}
```
# Configuration
**configure MySQL**
```bash
mysql
```
Login with your password you set earlier. <br />
Create new user
```MYSQL
CREATE USER 'solder'@'localhost' IDENTIFIED BY 'secret';
```
> **NOTE: By writing *IDENTIFIED BY 'secret'* you set your password. Dont use *secret***

<br />

Create database solder and grant user *solder* access to it.

```MYSQL
CREATE DATABASE solder;
GRANT ALL ON solder.* TO 'solder'@'localhost';
FLUSH PRIVILEGES;
exit
```

**Configure TechnicSolder** <br />

```bash
chown -R www-data html
```

Go to `http://your_server_IP_address` and fill out the form. If you followed these instructions, database name and username is `solder` <br />

The final step is to set your Solder URL in Solder Configuration (In your https://technicpack.net profile)

For some reason after filling this out Apache doesn't restart so restart apache one more time and you shoudl be up and running again!

```bash
service apache2 restart
```

That's it. You have successfully installed and configured TechnicSolder. It's ready to use!
