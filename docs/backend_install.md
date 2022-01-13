

## Package requirement:

```
apt install php7.4-fpm php7.4-mysql php7.4-xml nginx mariadb-server
```

???+ note
    To get FastCGI running for the nginx webservice, you will install the Debian package fcgiwrap

## MariaDB Configuration

```
mysql_secure_installation
```
```
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

After the installation, Login to mariadb as root.

```
mysql -u root -p
UPDATE mysql.user SET plugin = 'mysql_native_password' WHERE User = 'root';
FLUSH PRIVILEGES;
```

Create a database and user for GLPI.
```
CREATE DATABASE fusionsuite_db;
CREATE USER 'fusionsuite_user'@'localhost' IDENTIFIED BY 'StrongDBPassword';
GRANT ALL PRIVILEGES ON fusionsuite_db.* TO 'fusionsuite_user'@'localhost';
FLUSH PRIVILEGES;
```

## Download FusionSuite backend

```
mkdir /var/www/fusionsuite
git clone https://github.com/fusionSuite/backend.git
cd /var/www/fusionsuite
```

## Configure phinx with your mariadb parameters

```
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

Disable default site.

```bash
rm /etc/nginx/sites-available/default
```

create your own conf file for FusionSuite.

```bash
vim /etc/nginx/sites-available/fusionsuite.conf
```


Default configuration

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/fusionsuite/backend/public;
        index index.php index.html index.htm index.nginx-debian.html;

        server_name _;


        location / {
                try_files $uri $uri/ =404;
        }
        location ~ ^/(status|ping)$ {
                allow 127.0.0.1;
                fastcgi_param SCRIPT_FILENAME $document_root/index.php$fastcgi_script_name;
                fastcgi_index index.php;
                include fastcgi_params;
                fastcgi_pass   unix:/run/php/php7.4-fpm.sock;
        }
        location ~ \.php$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php7.4-fpm.sock; # PHP version (php -v command)
        }
}
```
