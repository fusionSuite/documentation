# Frontend installation

This documentation has been made and tested for:

- Debian 11 (Bullseye)
- Nginx

Other systems should work as well, but you may have to adapt some commands.

## Conventions used in this document

- commands starting by `#` must be executed by the `root` user;
- commands starting by `$` must be executed by your normal user.

## Install the package dependencies

Install the dependencies with the following command:

```console
# apt install curl nginx
```

If you want to install FusionSuite with Git (recommended method), you should
install it as well:

```console
# apt install git
```

## Install Node.js and Yarn

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

## Prepare the files and compile the code

Clone the repository in your home directory (this can be done as your normal
user):

```console
$ git clone https://github.com/fusionSuite/frontend.git ~/fusionsuite-frontend
```

??? tip "Not using Git?"
    Instead of using Git, you may prefer to get FusionSuite via an archive:

    ```console
    $ curl -L --output fusionsuite.tar.gz https://github.com/fusionSuite/frontend/archive/refs/heads/master.tar.gz
    $ tar xzf fusionsuite.tar.gz
    $ mv frontend-master/ ~/fusionsuite-frontend
    ```

Install the dependencies with Yarn:

```console
$ cd ~/fusionsuite-frontend
$ make install
```

And compile the frontend:

```console
$ make build
```

The command should have compiled the frontend to the `dist/frontend/` directory.
You can choose either the English version (`en-US`) or French version (`fr`).

We now want the webserver to serve these files. Please note the following
commands are executed as the `root` user.

In case you did not create this directory for the backend yet, create it now:

```console
# mkdir /var/www/fusionsuite
```

Move the compiled files to this directory:

```
# mv ~your-user/fusionsuite-frontend/dist/frontend/en-US /var/www/fusionsuite/frontend
```

And don't forget to set the correct permissions on the files:

```console
# chown -R www-data:www-data /var/www/fusionsuite
```

## Configure Nginx

Edit the file `/etc/nginx/sites-available/fusionsuite.conf` by adapting the
following example (especially the `server_name` and `proxy_pass` directives):

???+ note "/etc/nginx/sites-available/fusionsuite.conf"
    ```nginx
    server {
      listen 80;
      listen [::]:80;

      root /var/www/fusionsuite/frontend;
      index index.html;
      server_name fusionsuite.example.com;

      location / {
        try_files $uri $uri/ /index.html =404;
      }

      location /api/ {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://fusionsuite-backend.example.com;
      }
    }
    ```

!!! info
    Instead of proxifying `/api`, it is possible to tell the frontend to call
    the backend directly. Create a `src/config.json` file:

    ```console
    # cp ~your-user/fusionsuite-frontend/src/config.sample.json /var/www/fusionsuite/frontend/config.json
    # chown www-data:www-data /var/www/fusionsuite/frontend/config.json
    ```

    And adapt the `backendUrl` value.

!!! tip
    If you already have a block for the backend, you can put this one before or
    after, or you can create a different file under `/etc/nginx/sites-available`.

!!! tip
    You can check your configuration is correct with the command `nginx -t`.

If it's not already the case, enable the configuration file:

```console
# ln -s /etc/nginx/sites-available/fusionsuite.conf /etc/nginx/sites-enabled/fusionsuite.conf
```

Then reload Nginx:

```console
# systemctl reload nginx
```

## Test and validate

Now, the frontend should be accessible at the URL you've configured in Nginx
(at least if your DNS is correctly configured!)

You can open your browser at `http://fusionsuite.example.com` and verify
FusionSuite works.

If the connection to the backend works, you should see a bunch of types in the
menu on the left.

If the interface is displayed, but the menu is empty, you should verify the
`backendUrl` value in the file `/var/www/fusionsuite/frontend/config.json`.
