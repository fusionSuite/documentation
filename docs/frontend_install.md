This documentation has been made and tested for:  

- Debian 11 (Bullseye).  
- NodeJS
- yarn
- Nginx

## Package requirement

Make your system up to date and install the basics requirements.
```console
apt update && apt upgrade -y
apt install curl vim git -y
```

### NodeJs

Let's install nodeJs:
```console
curl -fsSL https://deb.nodesource.com/setup_current.x | bash -
apt update
apt install nodejs -y
```

### yarn

Let's install yarn
```console
curl -o- -L https://yarnpkg.com/install.sh | bash
```

### nginx

In case you did not installed nginx for the backend already, install it now:
```console
apt install nginx -y
```

## Prepare the files and compile the code

In case you did not create this directory for the backend already, create it now:
```console
mkdir /var/www/fusionsuite
```

Clone the repository in `/var/www/fusionsuite/`
```console
git clone https://github.com/fusionSuite/frontend.git /var/www/fusionsuite/frontend
```

Install yarn:
```console
cd /var/www/fusionsuite/frontend
yarn install
```

Compile:
```console
./node_modules/.bin/ionic build --prod -- --aot=true --buildOptimizer=true --optimization=true --vendor-chunk=true
```

Update the file `config.json` to point on our backend url
```console
vim /var/www/fusionsuite/frontend/www/config.json
```

!!! note "TODO"
    Add an example here

## Nginx configuration

??? tip "Tip: remove the default nginx website"
    If you want to disable the default nginx website just delete the file `/etc/nginx/sites-enabled/default`
    ````console
    rm /etc/nginx/sites-enabled/default
    ```
Configure nginx to point to folder `/var/www/fusionsuite/frontend/www/` with the following example:

???+ example "/etc/nginx/sites-available/fusionsuite.conf"
    ```nginx
    server {
      listen 80 default_server;
      listen [::]:80 default_server;

      root /var/www/fusionsuite/frontend/www;
      index index.php index.html;
      server_name _;

      location / {
        allow        127.0.0.1;
      }
    }
    ```

### Enable the configuration and start NGINX

??? tip "Tip: Check your configuration"
    You can check your configuration with the command `nginx -t`

If it not already the case (the backend is configured in the same file), make your site enable.
```console
ln -s /etc/nginx/sites-available/fusionsuite.conf /etc/nginx/sites-enabled/fusionsuite.conf
```

Then restart nginx
```console
systemctl restart nginx
```
