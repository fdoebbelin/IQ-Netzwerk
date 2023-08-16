---
source: https://idroot.us/install-suitecrm-ubuntu-22-04/
source: https://www.vultr.com/docs/how-to-install-suitecrm-on-ubuntu-20-04/
source: https://www.osradar.com/install-suitecrm-ubuntu-20-04/
source: https://www.linuxbabe.com/ubuntu/install-suitecrm-ubuntu-20-04-apache-nginx
---

## Downloading and installing SuiteCRM

If you are installing SuiteCRM for the first time, follow the instructions in this section. If you have an earlier version of SuiteCRM installed, refer to the upgrade section for instructions on how to upgrade your SuiteCRM instance. Follow the steps listed below to install SuiteCRM:


## Install LAMP-Stack
### Step 1.
First, make sure that all your system packages are up-to-date by running the following `apt` commands in the terminal.

```bash
sudo apt update && sudo apt upgrade
sudo apt install lsb-release ca-certificates apt-transport-https software-properties-common
```

### Step 2. 
Installing Apache HTTP Server on Ubuntu 22.04.

By default, the Apache is available on Ubuntu 22.04 base repository. Now run the following command below to install the latest version of Apache to your Ubuntu system:

```sh
sudo apt install apache2 apache2-utils
```

After successfully installation, enable Apache (to start automatically upon system boot), start, and verify the status using the commands below:

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
sudo systemctl status apache2
```

or for WSL

```bash
sudo service apache2 start
sudo service apache2 status
```

You can confirm the Apache2 version with the below command:

```bash
apache2 -v
```

### Step 3. 
Configure Firewall.

Now we set up an Uncomplicated Firewall (UFW) with Apache to allow public access on default web ports for HTTP and HTTPS:

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Apache Full'
sudo ufw enable
```

### Step 4. 
Accessing Apache Web Server.

```sh
curl http://localhost
```

### Step 5. 
Installing MariaDB on Ubuntu 22.04.

By default, the MariaDB is available on Ubuntu 22.04 base repository. Now run the following command below to install the latest version of MariaDB to your Ubuntu system:


```sh
sudo apt install mariadb-server
```

After successfully installation, enable MariaDB (to start automatically upon system boot), start, and verify the status using the commands below:


```sh
sudo systemctl enable mariadb
sudo systemctl start mariadb
sudo systemctl status mariadb
```

Confirm the installation and check the installed build version of MariaDB:

```
mariadb --version
mysql --version
sudo mysql -u root 
```

### Step 6. 
Secure MariaDB installation.

By default, MariaDB is not hardened. You can secure MariaDB using the `mysql_secure_installation` script. you should read and below each step carefully which will set a root password, remove anonymous users, disallow remote root login, and remove the test database and access to secure MariaDB:


```sh
sudo mysql_secure_installation
```

Configure it like this:

- Set root password? [Y/n] y
	- root
- Remove anonymous users? [Y/n] y
- Disallow root login remotely? [Y/n] y
- Remove test database and access to it? [Y/n] y
- Reload privilege tables now? [Y/n] y

You can now connect to the MariaDB server using the new password:


```
mysql -u root -proot
```

### Step 7. 
Installing PHP on Ubuntu 22.04.

By default, the PHP is not available on Ubuntu 22.04 base repository. Now run the following command below to add the Ondrej PPA to your system:


```
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```

After the repository was added, Update the APT index then install PHP 8.1 using the following command below:


```
sudo apt install php libapache2-mod-php php-mysql

oder

sudo apt install php8.2 php8.2-common libapache2-mod-php8.2 php8.2-cli php8.2-fpm php8.2-xml
```


Check PHP version information:


```
php --version
```

Output:

PHP 8.1.5 (cli) (built: May  20 2022 17:46:36) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.5, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.5, Copyright (c), by Zend Technologies

Most PHP applications depend on various extensions to extend their features. That can also be installed using the following syntax:


```
sudo apt install php8.2-[extension]
```

Replace **[extension]** with the extension you want to install, if you want to add multiple extensions then include them in braces, I am going to install **“php-mbstring, php-mysql, php-xml, and php-curl”** by running the below-mentioned command:

```
sudo apt install php8.2-mysql php8.2-mbstring php8.2-xml php8.2-curl 
```

Users who have installed different PHP versions, need to replace `8.2` with the required PHP versions.

## About PHP Configuration Files

The PHP configuration files are stored under **/etc/php** directory with the version numbers. For example, all the configuration files related to PHP 8.2 are located below:

1.  Main PHP configuration file location:
    -   PHP CLI: /etc/php/8.2/cli/php.ini
    -   Apache: /etc/php/8.2/apache2/php.ini
    -   PHP FPM: /etc/php/8.2/fpm/php.ini
2.  All the installed PHP modules are stored under **/etc/php/8.2/mods-available** directory.
3.  PHP Active modules configuration directory location:
    -   PHP CLI: /etc/php/8.2/cli/conf.d/
    -   Apache: /etc/php/8.2/apache2/conf.d/
    -   PHP FPM: /etc/php/8.2/fpm/conf.d/

To check files for the other PHP versions, just change the PHP version number (8.2 in the above example) in the files and directory path.

### Step 8. 
Create Apache Virtualhost.

First, create a root directory to hold your website’s files:


```sh
sudo mkdir -p /var/www/html/domain.com/
```


Then, change the ownership and group of the directory:


```sh
sudo chown -R www-data:www-data /var/www/html/domain.com/
```

After that, we create an Apache virtual host to serve the HTTP version of the website:


```sh
sudo nano /etc/apache2/sites-available/www.domain.com.conf
```

Add the following file:

```
<VirtualHost *:80>
   ServerName domain.com
   ServerAlias www.domain.com
   ServerAdmin admin@domain.com
   DocumentRoot /var/www/html/www.domain.com

   ErrorLog ${APACHE_LOG_DIR}/www.domain.com_error.log
   CustomLog ${APACHE_LOG_DIR}/www.domain.com_access.log combined

   <Directory /var/www/html/www.domain.com>
      Options FollowSymlinks
      AllowOverride All
      Require all granted
   </Directory>
</VirtualHost>
```


Save and close the file, then restart the Apache webserver so that the changes take place:


```sh
sudo a2ensite www.domain.com.conf
sudo a2enmod ssl rewrite
sudo systemctl restart apache2
```

### 3.- Configuring PHP to work with SuiteCRM

PHP is a very flexible language and we can configure it together with Apache to increase its power. In addition to this, the process is necessary to enjoy a full experience with SuiteCRM

So, open the configuration file:

sudo nano /etc/php/7.4/apache2/php.ini

And using the key combination **CTRL + W** edit the following fields and assign these values:

file_uploads = On
default_charset = UTF-8
memory_limit = 128M
post_max_size = 60M
upload_max_filesize = 60M
memory_limit = 256M
max_input_time = 60
max_execution_time = 5000

Then save the changes, close the editor, and restart Apache to apply the changes.

sudo systemctl restart apache2

### Step 9. 
Secure Apache with Let’s Encrypt on Ubuntu 22.04.
[How To Install LAMP Stack on Ubuntu 22.04 LTS - idroot](https://idroot.us/install-lamp-stack-ubuntu-22-04/)

### Step 10. 
Auto-Renewal SSL.
[How To Install LAMP Stack on Ubuntu 22.04 LTS - idroot](https://idroot.us/install-lamp-stack-ubuntu-22-04/)

### Step 11. 
Test PHP.

To test PHP scripts we need to add the `info.php` file in the document:

```sh
nano /var/www/html/www.domain.com/info.php
```

Add the following to the file:

```php
<?php phpinfo(); ?>
```

Let’s make sure that the server correctly displays the content generated by the PHP script by opening this page in the browser: `https://www.domain.com/info.php`


## Download the SuiteCRM files
Download and extract

```bash
wget -O suitecrm.zip https://suitecrm.com/download/128/suite82/562059/suitecrm-8-2-4.zip
sudo unzip suitecrm.zip -d /var/www/
sudo mv /var/www/SuiteCRM-7.12.5/ /var/www/suitecrm
```

Change folder permission

```bash
cd /var/www/suitecrm
sudo chown -R www-data:www-data /var/www/suitecrm/
sudo chmod -R 755 .
sudo chmod -R 775 cache custom modules themes data upload
sudo chmod 775 config_override.php 2>/dev/null
```
    
## Install SuiteCRM