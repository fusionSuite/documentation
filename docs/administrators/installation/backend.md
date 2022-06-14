# Backend installation

This documentation has been made and tested for:

- Debian 11 (Bullseye)
- MariaDB
- Nginx

Other systems should work as well, but you may have to adapt some commands.

FusionSuite also works and is tested with MySQL and PostgreSQL databases.

## Conventions used in this document

- commands starting by `#` must be executed by the `root` user;
- commands starting by `$` must be executed by your normal user.

## Install the package dependencies

Install the dependencies with the following command:

```console
# apt install curl composer php7.4-fpm php7.4-mysql php7.4-xml nginx mariadb-server
```

If you want to install FusionSuite with Git (recommended method), you should
install it as well:

```console
# apt install git
```

## Configure MariaDB

Let's start by securing MariaDB a little bit:

```console
# mariadb-secure-installation
```

??? example "Output example "
    ```console
    NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
        SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

    In order to log into MariaDB to secure it, we'll need the current
    password for the root user. If you've just installed MariaDB, and
    haven't set the root password yet, you should just press enter here.

    Enter current password for root (enter for none):

    Set root password? [Y/n] Y
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

Login to MariaDB as root (with the password you've just set):

```console
# mariadb -u root -p
```

Then, create a database and a user for FusionSuite (you must adapt the commands
with your own credentials):

```mysql
CREATE DATABASE fusionsuite_production;
CREATE USER 'fusionsuite_user'@'localhost' IDENTIFIED BY 'StrongDBPassword';
GRANT ALL PRIVILEGES ON fusionsuite_production.* TO 'fusionsuite_user'@'localhost';
FLUSH PRIVILEGES;
```

## Install and configure the backend

Clone the backend repository at a location where the `www-data` user has access
(e.g. `/var/www/fusionsuite`):

```console
# mkdir /var/www/fusionsuite
# git clone https://github.com/fusionSuite/backend.git /var/www/fusionsuite/backend
# chown -R www-data:www-data /var/www/fusionsuite
```

??? tip "Not using Git?"
    Instead of using Git, you may prefer to deploy FusionSuite via an archive:

    ```console
    # mkdir /var/www/fusionsuite
    # curl -L --output fusionsuite.tar.gz https://github.com/fusionSuite/backend/archive/refs/heads/master.tar.gz
    # tar xzf fusionsuite.tar.gz
    # mv backend-master/ /var/www/fusionsuite/backend
    # chown -R www-data:www-data /var/www/fusionsuite
    ```

!!! info
    The next commands are executed by the `www-data` user to make sure that all
    files are accessible to the user who run the webserver.

In the `backend/` directory, install the composer dependencies:

```console
# cd /var/www/fusionsuite/backend
# sudo -u www-data composer install
```

Create an environment configuration file for production:

```console
# sudo -u www-data ./bin/cli env:create -c \
    -n production \
    -t MariaDB \
    -H localhost \
    -d fusionsuite_production \
    -u fusionsuite_user \
    -p StrongDBPassword \
    -P 3306
```

Finally, setup the database:

```console
# sudo -u www-data ./bin/cli install
```

## Configure Nginx

Create and edit the file `/etc/nginx/sites-available/fusionsuite.conf` by
adapting the following example (especially the `server_name` directive):

???+ note "/etc/nginx/sites-available/fusionsuite.conf"
    ```nginx
    server {
      listen            80;
      listen            [::]:80;

      root              /var/www/fusionsuite/backend/public;
      index             index.php index.html;
      server_name       fusionsuite-backend.example.com;

      location / {
        fastcgi_param   SCRIPT_FILENAME $document_root/index.php$fastcgi_script_name;
        include         fastcgi_params;
        fastcgi_pass    unix:/run/php/php7.4-fpm.sock;

        add_header      Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
        add_header      X-Frame-Options "SAMEORIGIN";
        add_header      Access-Control-Allow-Origin *;
        add_header      Access-Control-Allow-Methods 'GET, POST, OPTIONS, PUT, DELETE';
        add_header      Access-Control-Allow-Credentials true;
        add_header      Access-Control-Allow-Headers 'Origin,Content-Type,Accept,Authorization,Cache-Control,Pragma,Expires';
        add_header      Access-Control-Expose-Headers 'X-Total-Count,Content-Range,Link';
      }
    }
    ```

!!! tip
    You can check your configuration is correct with the command `nginx -t`.

Enable the configuration by making a symbolic link under `/etc/nginx/sites-enabled/`
 and reload Nginx to apply the configuration:

```console
# ln -s /etc/nginx/sites-available/fusionsuite.conf /etc/nginx/sites-enabled/fusionsuite.conf
# systemctl reload nginx
```

## Test and validate

Now, your backend should be accessible at the URL you've configured in Nginx
(at least if your DNS is correctly configured!)

The request to the URL `http://fusionsuite-backend.example.com` should answer
something like this:

```json
{"status":"error","message":"Token not found."}
```

at `http://fusionsuite-backend.example.com/ping`:

```console
pong
```

at `http://fusionsuite-backend.example.com/v1/status`:

```json
{"connections":{"database":true}}
```

!!!tip
    If the answer is:

    ```json
    {"connections":{"database":false}}
    ```

    Please check:

    - your database configuration in `/var/www/fusionsuite/backend/config/current/database.php`
    - the MariaDB status and logs with `systemctl status mariadb` and `journalctl -u mariadb`
