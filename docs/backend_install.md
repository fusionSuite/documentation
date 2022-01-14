
---
Author: Maël  
Date: 2022.01.14  
---

!!! note
    This documentation has been made and tested for Debian 11 (Bullseye).  
    Documentation for other distribution will come soon.

## Package requirement:

```console
apt install php7.4-fpm php7.4-mysql php7.4-xml nginx mariadb-server
```

???+ tip
    To get FastCGI running for the nginx webservice, you will install the Debian package fcgiwrap

## MariaDB Configuration

### Let's secure a little bit MariaDB
```console
mysql_secure_installation
```

??? example "Output example "
    ```console
    NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
        SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

    In order to log into MariaDB to secure it, we'll need the current
    password for the root user. If you've just installed MariaDB, and
    haven't set the root password yet, you should just press enter here.

    Enter current password for root (enter for none):

    Set root password? [Y/n]
    New password:
    Re-enter new password:
    Password updated successfully!
    Reloading privilege tables..
    ... Success!
    [...]
    Remove anonymous users? [Y/n] Y
    [...]
    Disallow root login remotely? [Y/n] Y
    [...]
    Remove test database and access to it? [Y/n] Y
    [...]
    Reload privilege tables now? [Y/n] Y
    [...]
    ```

### Login to mariadb as root.

```console
mysql -u root -p
```

### Create a database and user for FusionSuite.

Please adapt with your personals credentials.
```mysql
CREATE DATABASE fusionsuite_db;
CREATE USER 'fusionsuite_user'@'localhost' IDENTIFIED BY 'StrongDBPassword';
GRANT ALL PRIVILEGES ON fusionsuite_db.* TO 'fusionsuite_user'@'localhost';
FLUSH PRIVILEGES;
```

## Download FusionSuite backend

```console
mkdir /var/www/fusionsuite
cd /var/www/fusionsuite
git clone https://github.com/fusionSuite/backend.git
```

## Configure phinx with your mariadb parameters

Edit the file: `/var/www/fusionsuite/backend/phinx.php`

Modify the line 27 according your needs.  
For production, replace:  

```php
'default_environment' => 'development',
```

by:  
```php
'default_environment' => 'production',
```

Then edit this part according your configuration:  
```php
'production' => [
    'adapter' => 'mysql',
    'host' => 'localhost',
    'name' => 'fusionsuite_db',
    'user' => 'fusionsuite_user',
    'pass' => '',
    'port' => '3306',
    'charset' => 'utf8',
]
```

## Nginx Configuration

### Disable default site
```console
rm /etc/nginx/sites-available/default
```

### Create your own config file for FusionSuite

Create and edit the file `/etc/nginx/sites-available/fusionsuite.conf` with the following example.

```nginx
server {
  listen 80 default_server;
  listen [::]:80 default_server;

  root /var/www/fusionsuite/backend/public;
  index index.php index.html;
  server_name _;

  location / {
    try_files $uri $uri/ =404;
  }

  location ~ \.php$ {
    include		snippets/fastcgi-php.conf;
    fastcgi_pass	unix:/run/php/php7.4-fpm.sock;
  }

  location ~ ^/(status|ping)$ {
    allow		127.0.0.1;
    fastcgi_param	SCRIPT_FILENAME $document_root/index.php$fastcgi_script_name;
    include		fastcgi_params;
    fastcgi_pass	unix:/run/php/php7.4-fpm.sock;
  }
}
```

### Enable this configuration and test

```
ln -s /etc/nginx/sites-available/fusionsuite.conf /etc/nginx/sites-enabled/fusionsuite.conf
systemctl restart nginx
```
