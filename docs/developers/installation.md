# Installation of the development environment

This document explains how to install your development environment. It can be
setup [with](#with-docker) or [without](#without-docker) Docker, depending on
your preferences.

!!! info
    This document uses the repositories of the [FusionSuite GitHub organization](https://github.com/fusionSuite).
    If you plan to contribute to the project, you should prefer to fork the
    backend and frontend repositories and adapt the `git clone` commands with
    your forks.

## Conventions used in this document

- commands starting by `#` must be executed by the `root` user;
- commands starting by `$` must be executed by your normal user.

## Without Docker

This guide has been made and tested for:

- Debian 11 (Bullseye)
- MariaDB
- Nginx

Other systems should work as well, but you may have to adapt some commands.

FusionSuite also works and is tested with MySQL and PostgreSQL databases.

### Install the package dependencies

Install the dependencies with the following command:

```console
# apt install git curl composer php8.0-fpm php8.0-mysql php8.0-xml nginx mariadb-server
```

### Configure MariaDB

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

### Install and configure the backend

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
$ make install
```

Setup your system (i.e. create a configuration file and initialize your
database):

```console
$ make setup
```

This command accepts few parameters, for instance (these are the default
values):

```
$ make setup ENV_NAME=development \
    DB_TYPE=MariaDB \
    DB_HOST=localhost \
    DB_NAME=fusionsuite_development \
    DB_USER=fusionsuite \
    DB_PASS=fusionsuite \
    DB_PORT=3306
```

### Configure Nginx

You now have to configure Nginx to serve the backend. Create a new file named
`/etc/nginx/sites-available/fusionsuite.conf`:

???+ note "/etc/nginx/sites-available/fusionsuite.conf"
    ```nginx
    server {
        listen 8000;
        listen [::]:8000;

        server_name localhost;
        root /var/www/fusionsuite/backend/public;
        index index.html index.php;

        location / {
            fastcgi_param   SCRIPT_FILENAME $document_root/index.php$fastcgi_script_name;
            include         fastcgi_params;
            fastcgi_pass    unix:/run/php/php8.0-fpm.sock;

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

!!! info
    By default, in development, the frontend expects to find the backend at
    `localhost:8000`. It is also possible to specify a custom URL for the
    backend, it will be explained how below.

!!! tip
    You can check your configuration is correct with the command `nginx -t`.

Then, enable this configuration:

```console
# ln -s /etc/nginx/sites-available/fusionsuite.conf /etc/nginx/sites-enabled/fusionsuite.conf
# systemctl reload nginx
```

The backend should now be accessible at [localhost:8000](http://localhost:8000).

You can check the database is correctly configured at [localhost:8000/v1/status](http://localhost:8000/v1/status).
If it’s correct, it will return the following Json:

```json
{"connections":{"database":true}}
```

### Install and configure the frontend

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
$ make install
```

!!! info
    If you've configured the backend at a different location than `localhost:8000`,
    create a `src/config.json` file:

    ```console
    $ cp src/config.sample.json src/config.json
    ```

    And adapt the `backendUrl` value.

Finally, start the frontend:

```console
$ make start
```

When it's ready, the frontend should be accessible at [localhost:4200](http://localhost:4200).

If everything is fine, you should see a login form. The default credentials
are: `admin` / `admin`.

## With Docker

If you prefer to setup your environment with Docker, please make sure to
[install Docker Engine](https://docs.docker.com/engine/install/) and [Docker
Compose](https://docs.docker.com/compose/install/). Both `docker` and
`docker-compose` must be executable by your normal user.

!!! warning
    If you're not used to Docker, we recommend you to install FusionSuite [the
    non-Docker way](#without-docker): the setup may look simpler, but Docker
    comes with its own difficulties.

### Setup the backend

Clone the backend repository:

```console
$ git clone https://github.com/fusionSuite/backend.git
$ cd backend
```

Install the dependencies with:

```console
$ make install DOCKER=true
```

!!! info
    This first command builds the Docker image for PHP, and so it can take
    some time to finish. Don't worry: once done, the image don't need to be
    rebuilt.

!!! tip
    You can declare the `DOCKER` environment variable in your e.g. `.bashrc`
    file to not have to pass it to each command.

Start the Docker containers with:

```console
$ make docker-start
```

!!! info
    This second command downloads the images for MariaDB and Nginx, which can
    also be long.

In a different console, setup your system with:

```console
$ make setup DOCKER=true
```

The backend should now be accessible at [localhost:8000](http://localhost:8000).

You can check the database is correctly configured at [localhost:8000/v1/status](http://localhost:8000/v1/status).
If it’s correct, it will return the following Json:

```json
{"connections":{"database":true}}
```

### Setup the frontend

Clone the frontend repository:

```console
$ git clone https://github.com/fusionSuite/frontend.git
$ cd frontend
```

Install the dependencies with:

```console
$ make install DOCKER=true
```

!!! info
    This command builds the Docker image for NodeJs. Same as before: it might
    be long, but it's only the first time.

Start the Docker container with:

```console
$ make docker-start
```

When it's ready, the frontend should be accessible at [localhost:4200](http://localhost:4200).

If everything is fine, you should see a login form. The default credentials
are: `admin` / `admin`.

### Work in the Docker containers

In a Docker environment, you can't execute the commands directly (e.g. `php`,
`mariadb`, `npm`…): you need to run them _in_ the containers.

Hopefully, we provide some commands wrappers to facilitate your life. You'll
find them under the `docker/bin` folders. You can use them as you would use the
normal commands.

Backend repository:

```console
$ ./docker/bin/php
$ ./docker/bin/cli
$ ./docker/bin/composer
$ ./docker/bin/mariadb
```

Frontend repository:

```console
$ ./docker/bin/ng
$ ./docker/bin/npm
$ ./docker/bin/yarn
```
