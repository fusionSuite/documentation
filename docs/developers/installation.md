# Installation of the development environment

This document explains how to install your development environment. It has been
made and tested for:

- Debian 11 (Bullseye)
- MariaDB
- Nginx

Other systems should work as well, but you may have to adapt some commands.

FusionSuite also works and is tested with MySQL and PostgreSQL databases.

!!! info
    This document uses the repositories of the [FusionSuite GitHub organization](https://github.com/fusionSuite).
    If you plan to contribute to the project, you should prefer to fork the
    backend and frontend repositories and adapt the `git clone` commands with
    your forks.

## Conventions used in this document

- commands starting by `#` must be executed by the `root` user;
- commands starting by `$` must be executed by your normal user.

## Install the package dependencies

Install the dependencies with the following command:

```console
# apt install git curl composer php7.4-fpm php7.4-mysql php7.4-xml nginx mariadb-server
```

## Configure MariaDB

!!! warning
    Please note the instructions in this section don't secure the access to
    MariaDB. If you need it, please follow [the instructions for administrators](../administrators/installation/backend.md#configure-mariadb).

Login to MariaDB with the `root` user (it has no passwords by default):

```console
# mariadb -u root
```

In this shell, create a new database for FusionSuite. The following commands
create a database named `fusionsuite_development` and create a user
`fusionsuite` (with password `fusionsuite`) with full access to it:

```mysql
CREATE DATABASE fusionsuite_development;
CREATE USER 'fusionsuite'@'localhost' IDENTIFIED BY 'fusionsuite';
GRANT ALL PRIVILEGES ON fusionsuite_development.* TO 'fusionsuite'@'localhost';
FLUSH PRIVILEGES;
```

## Install and configure the backend

Clone the backend repository at a location where you and the `www-data` user
have access (e.g. `/var/www/fusionsuite`):

```console
# mkdir /var/www/fusionsuite
# chown 1000:1000 /var/www/fusionsuite
$ # Then, with your normal user:
$ git clone https://github.com/fusionSuite/backend.git /var/www/fusionsuite/backend
```

!!! info
    You might have to adapt the second command with the user and group ids of
    your normal user.

Then, in the `backend/` directory, install the composer dependencies:

```console
$ cd /var/www/fusionsuite/backend
$ composer install
```

Create an environment configuration file for development:

```console
$ ./bin/cli env:create -c \
    -n development \
    -t MariaDB \
    -H localhost \
    -d fusionsuite_development \
    -u fusionsuite \
    -p fusionsuite \
    -P 3306
```

Finally, setup the database:

```console
$ ./bin/cli install
```

## Configure Nginx

You now have to configure Nginx to serve the backend. Create a new file named
`/etc/nginx/sites-available/fusionsuite.conf`:

???+ note "/etc/nginx/sites-available/fusionsuite.conf"
    ```nginx
    server {
        listen 80;
        listen [::]:80;

        server_name fusion-backend.localhost;
        root /var/www/fusionsuite/backend/public;
        index index.html index.php;

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

Then, enable this configuration:

```console
# ln -s /etc/nginx/sites-available/fusionsuite.conf /etc/nginx/sites-enabled/fusionsuite.conf
# systemctl reload nginx
```

The backend should now be accessible at [fusion-backend.localhost](http://fusion-backend.localhost).

You can check the database is correctly configured at [fusion-backend.localhost/v1/status](http://fusion-backend.localhost/v1/status).
If it’s correct, it will return the following Json:

```json
{"connections":{"database":true}}
```

## Install and configure the frontend

Clone the frontend repository next to the backend repository:

```console
$ git clone https://github.com/fusionSuite/frontend.git /var/www/fusionsuite/frontend
```

Install [`nvm`](https://github.com/nvm-sh/nvm) so you can easily install and
manage different versions of Node.js:

```console
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
$ source ~/.bashrc
$ nvm install 16
```

!!!tip
    The last command installs the version 16 of Node.js. You can replace `16`
    by `node` to get the latest version, or by a specific version if you need
    it.

Then, install Yarn:

```console
$ npm install --location=global yarn
```

Install the Node dependencies:

```console
$ cd /var/www/fusionsuite/frontend
$ yarn install
```

Edit the `src/config.json` file and change the `backendUrl` value with your
backend endpoint:

???+ note "/var/www/fusionsuite/frontend/src/config.json"
    ```json
    {
      "backendUrl": "http://fusion-backend.localhost"
    }
    ```

Finally, start the frontend:

```console
$ ./node_modules/.bin/ionic serve
```

When it's ready, it should open your browser at [localhost:8100](http://localhost:8100).

If everything is fine, you should see a bunch of types in the menu on the left.
