# install frontend

* NodeJs

```apt-get update && apt-get upgrade -y```

```apt-get install curl -y```

```curl -fsSL https://deb.nodesource.com/setup_current.x | bash -```

```apt-get update ```

```apt-get install nodejs -y```


* yarn

```curl -o- -L https://yarnpkg.com/install.sh | bash```

* nginx

```apt-get install nginx -y```

* git

```apt-get install git -y```

* create directory

```mkdir /var/www/fusionsuite```

* Clone repository

```git clone https://github.com/fusionSuite/frontend.git /var/www/fusionsuite/frontend```

* install yarn

```cd /var/www/fusionsuite/frontend```

```yarn install```

```text
yarn install v1.22.17
[1/4] Resolving packages...
[2/4] Fetching packages...
[3/4] Linking dependencies...
warning " > @ionic-native/core@5.36.0" has incorrect peer dependency "rxjs@^5.5.0 || ^6.5.0".
warning " > @ionic-native/splash-screen@5.36.0" has incorrect peer dependency "rxjs@^5.5.0 || ^6.5.0".
warning " > @ionic-native/status-bar@5.36.0" has incorrect peer dependency "rxjs@^5.5.0 || ^6.5.0".
warning "@ionic/angular-toolkit > @angular-devkit/build-angular@12.2.14" has incorrect peer dependency "@angular/compiler-cli@^12.0.0".
warning "@ionic/angular-toolkit > @angular-devkit/build-angular@12.2.14" has incorrect peer dependency "typescript@~4.2.3 || ~4.3.2".
warning "@ionic/angular-toolkit > copy-webpack-plugin@9.1.0" has unmet peer dependency "webpack@^5.1.0".
warning "@ionic/angular-toolkit > @angular-devkit/build-angular > @ngtools/webpack@12.2.14" has incorrect peer dependency "@angular/compiler-cli@^12.0.0".
warning "@ionic/angular-toolkit > @angular-devkit/build-angular > @ngtools/webpack@12.2.14" has incorrect peer dependency "typescript@~4.2.3 || ~4.3.2".
warning " > codelyzer@6.0.2" has incorrect peer dependency "@angular/compiler@>=2.3.1 <13.0.0 || ^12.0.0-next || ^12.1.0-next || ^12.2.0-next".
warning " > codelyzer@6.0.2" has incorrect peer dependency "@angular/core@>=2.3.1 <13.0.0 || ^12.0.0-next || ^12.1.0-next || ^12.2.0-next".
[4/4] Building fresh packages...
Done in 131.48s.
```

* compile

```./node_modules/.bin/ionic build --prod -- --aot=true --buildOptimizer=true --optimization=true --vendor-chunk=true```

```text
> ng run app:build:production --aot=true --buildOptimizer=true --optimization=true --vendor-chunk=true
Node.js version v17.3.1 detected.
Odd numbered Node.js versions will not enter LTS status and should not be used for production. For more information, please see https://nodejs.org/en/about/releases/.
⠙ Generating browser application bundles (phase: setup)...Processing legacy "View Engine" libraries:
- @ionic-native/core [module/esm5] (https://github.com/ionic-team/ionic-native.git)
- @ionic-native/splash-screen [module/esm5] (https://github.com/ionic-team/ionic-native.git)
- @ionic-native/status-bar [module/esm5] (https://github.com/ionic-team/ionic-native.git)
Encourage the library authors to publish an Ivy distribution.
✔ Browser application bundle generation complete.
✔ Copying assets complete.
⠋ Generating index html...10 rules skipped due to selector errors:
  :host-context([dir=rtl]) .ion-float-start -> subselects_1.subselects[name] is not a function
  :host-context([dir=rtl]) .ion-float-end -> subselects_1.subselects[name] is not a function
  :host-context([dir=rtl]) .ion-float-sm-start -> subselects_1.subselects[name] is not a function
  :host-context([dir=rtl]) .ion-float-sm-end -> subselects_1.subselects[name] is not a function
  :host-context([dir=rtl]) .ion-float-md-start -> subselects_1.subselects[name] is not a function
  :host-context([dir=rtl]) .ion-float-md-end -> subselects_1.subselects[name] is not a function
  :host-context([dir=rtl]) .ion-float-lg-start -> subselects_1.subselects[name] is not a function
  :host-context([dir=rtl]) .ion-float-lg-end -> subselects_1.subselects[name] is not a function
  :host-context([dir=rtl]) .ion-float-xl-start -> subselects_1.subselects[name] is not a function
  :host-context([dir=rtl]) .ion-float-xl-end -> subselects_1.subselects[name] is not a function
✔ Index html generation complete.
....
```

* update config.json to point to backend url

```vim /var/www/fusionsuite/frontend/www/config.json```

* configure nginx to point to folder /var/www/fusionsuite/frontend/www/

```rm /etc/nginx/sites-enabled/default```

```vim /etc/nginx/sites-available/fusionsuite.conf```

example

```text
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

* enable the configuration and start NGINX

```ln -s /etc/nginx/sites-available/fusionsuite.conf /etc/nginx/sites-enabled/fusionsuite.conf```

```service nginx start```
