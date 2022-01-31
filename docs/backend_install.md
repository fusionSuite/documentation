
This documentation has been made and tested for:  

- Debian 11 (Bullseye).  
- MariaDB
- Nginx

However other distributions, web server and database engine are compatible as well.  

- Apache2
- MySQL
- PostgreSQL
- SQLite

## Package requirement


```console
apt install php7.4-fpm php7.4-mysql php7.4-xml nginx mariadb-server fcgiwrap
```

???+ note
    `fcgiwrap` seems to be a Debian specificity to get FastCGI running for the nginx webservice.

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
mkdir -p /var/www/fusionsuite
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

## Install dependencies and apply phinx configuration

!!! warn
    This part must be removed in the future because the tarball will be already prepare

Install composer:
```console
apt install composer -y
```

Install dependencies
```console
cd /var/www/fusionsuite/backend
composer install
```

Then apply phinx config with:
```console
./vendor/bin/phinx migrate
```

## Nginx Configuration

### Disable default site

The nginx example configuration can be deactivate with:
```console
rm /etc/nginx/sites-enabled/default
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

  add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
  add_header X-Frame-Options "SAMEORIGIN";
  add_header Access-Control-Allow-Origin *;
  add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS, PUT, DELETE';
  add_header Access-Control-Allow-Credentials true;
  add_header Access-Control-Allow-Headers 'Origin,Content-Type,Accept,Authorization,Cache-Control,Pragma,Expires';

  location / {
    allow        127.0.0.1;
    fastcgi_param    SCRIPT_FILENAME $document_root/index.php$fastcgi_script_name;
    include        fastcgi_params;
    fastcgi_pass    unix:/run/php/php7.4-fpm.sock;
  }
}
```

!!! note "TODO"
    Add here an example for SSL

### Enable the configuration and restart NGINX

Enable the configuration by making a symbolic link in `sites-enabled`.
```console
ln -s /etc/nginx/sites-available/fusionsuite.conf /etc/nginx/sites-enabled/fusionsuite.conf
```

Restart Nginx to apply your new configuration.
```console
systemctl restart nginx
```

??? tip
     It is also possible to just reload the configuration with:  
     ```console
     systemctl reload nginx
     ```

## Test and validation

Now your backend should works.  

The request on url: `http://my_server.my_domain.tld/` should answer something like this:
```json
{"status":"error","message":"Token not found."}
```

on: `http://my_server.my_domain.tld/ping`:  
```console
pong
```

on: `http://my_server.my_domain.tld/v1/status`:  
```json
{"connections":{"database":true}}
```

if the answer is:
```json
{"connections":{"database":false}}
```

Please check:

- Your databases parameters in `/var/www/fusionsuite/backend/phinx.php`.  
- MariaDB status with `systemctl status mariadb`
