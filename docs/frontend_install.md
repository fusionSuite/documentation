This documentation has been made and tested for:  

- Debian 11 (Bullseye).  
- NodeJS
- yarn
- Nginx

## Package requirement


```console
apt-get update && apt-get upgrade -y
apt-get install curl vim git -y 
```

## NodeJs

```console
curl -fsSL https://deb.nodesource.com/setup_current.x | bash -
apt-get update
apt-get install nodejs -y
```

## yarn

```console
curl -o- -L https://yarnpkg.com/install.sh | bash
```

 ## nginx

```console
apt-get install nginx -y
```

### Let's configure nginx

### _create directory_

```console
mkdir /var/www/fusionsuite
```

 ### _Clone repository_

```console
git clone https://github.com/fusionSuite/frontend.git /var/www/fusionsuite/frontend
```

### _install yarn_

```console
cd /var/www/fusionsuite/frontend
yarn install
```

### _compile_

```console
./node_modules/.bin/ionic build --prod -- --aot=true --buildOptimizer=true --optimization=true --vendor-chunk=true
```

### _update config.json to point to backend url_

```console
vim /var/www/fusionsuite/frontend/www/config.json
```

### _configure nginx to point to folder /var/www/fusionsuite/frontend/www/_

```console
rm /etc/nginx/sites-enabled/default
vim /etc/nginx/sites-available/fusionsuite.conf
```

example

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

### _enable the configuration and start NGINX_

```console
ln -s /etc/nginx/sites-available/fusionsuite.conf /etc/nginx/sites-enabled/fusionsuite.conf
service nginx start 
```