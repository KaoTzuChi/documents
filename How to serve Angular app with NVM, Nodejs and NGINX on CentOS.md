# How to serve Angular app with node version manager(NVM), Nodejs and NGINX on CentOS
## Version
- nvm 0.35.0
- npm 6.12.0
- Node 12.13.0
- NGINX 1.17.5
- CentOS 7

## Instal and configure node version manager(NVM)
Download and autoinstall nvm.
> $ mkdir /home/download

> $ cd /home/download

> $ wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.0/install.sh | bash

Append nvm source string to /root/.bashrc
> $ vi ~/.bashrc

Insert the code below into .bashrc.
```
export NVM_DIR=""$HOME/.nvm""
[ -s ""$NVM_DIR/nvm.sh"" ] && \. ""$NVM_DIR/nvm.sh""  # This loads nvm
[ -s ""$NVM_DIR/bash_completion"" ] && \. ""$NVM_DIR/bash_completion""  # This loads nvm bash_completion
export NVM_NODEJS_ORG_MIRROR=http://nodejs.org/dist  # This resolves the error: nvm ls-remote command results in N/A
```

The new settings of .bashrc will be applied after logout and relogin centOS. Without relogin action, we can apply it immediately by the commands below.
> $ source ~/.bashrc

or
> $ . ~/.bashrc

To verify that nvm has been installed:
> $ command -v nvm

or
> $ nvm --version

Check the versions of Node.js can be installed in the remote repository.
> $ nvm ls-remote

or
> $ nvm ls-remote --lts

Now we can install several Node.js versions as we need.
> $ nvm install v12.6.0

> $ nvm install v12.13.0

Check all installed Node.js versions.
> $ nvm list

To active a specific version, using "nvm use <version>".
> $ nvm use v12.13.0

To set a specific version as default after reboot, using "nvm alias default <version>".
> $ nvm alias default v12.13.0


## Install packages and build project
Active the Node.js version we need.
> $ nvm use v12.13.0

Go to the folder we want to keep the angular project, and build a new project.
> $ cd /www/mysite

> $ ng new mynodeapp

Go to the angular project folder, and install the packages we need by npm.
> $ cd /www/mysite/mynodeapp

> $ npm install -g npm

> $ npm install -g typescript

> $ npm install -g @angular/cli

> $ npm install --save express

> ...


If the source code is already done in other place, we can just copy it to "/www/mysite/mynodeapp/src".
> $ mv /www/mysite/mynodeapp/src /www/mysite/mynodeapp/src.bk

> $ cp -Rp /home/sourcecode/angular/src /www/mysite/mynodeapp

> $ /www/mysite/mynodeapp/ng build --prod


Now we can start ng to verify the app. 
> $ /www/mysite/mynodeapp/ng serve --host server_domain_or_IP --port 4200


## Serve by Node.js and Express web framework
Make sure we have installed express by npm. If we don't have the file "server.js" in the angular project folder, create a new one.
> $ vi /www/mysite/mynodeapp/server.js

Then insert the code below into "server.js".
```
// Get dependencies
const express = require('express');
const path = require('path');
const http = require('http');
const app = express();

// Point static path to dist
app.use(express.static(path.join(__dirname, 'dist/mynodeapp')));

// Catch all other routes and return the index file
app.get('*', (req, res) => {
  res.sendFile(path.join(__dirname, 'dist/mynodeapp/index.html'));
});

// Get port from environment and store in Express.
const port = process.env.PORT || '4200';
const host = 'server_domain_or_IP';
app.set('port', port);
app.set('host', host);

// Create HTTP server.
const server = http.createServer(app);

// Listen on provided port, on all network interfaces.
server.listen(port, host, () => console.log(`Node server is running on ${host}:${port}`));
```

Then we can run the app by Node.js.
> $ cd /www/mysite/mynodeapp

> $ ng build --prod

> $ node server.js


## Make the Node.js server as a system control service
Create a .service file for saving the directives of Node.js execution.
> $ vi /etc/systemd/system/mynode.service

Then insert the directives below into mynode.service.
```
[Unit]
Description=Node v12.13.0 for angular app
After=syslog.target
[Service]
ExecStart=/root/.nvm/versions/node/v12.13.0/bin/node /www/mysite/mynodeapp/server.js
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=nodejs-example
Environment=NODE_ENV=production
[Install]
WantedBy=multi-user.target
```

Now we can use mynode by the following commands
> $ systemctl status mynode

> $ systemctl start mynode

> $ systemctl stop mynode

> $ systemctl restart mynode

> $ systemctl enable mynode

> $ systemctl disable mynode

To reload mynode.service:
> $ systemctl daemon-reload

To check logs:
> $ journalctl -u mynode


## Confirm NGINX is installed
Check the version of NGINX and the service status.
> $ nginx -v

> $ systemctl status nginx

The installation of NGINX please reference [here](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/).


Make a log folder and create a .conf file for nginx.
> $ mkdir /www/mysite/mynodeapp/log

> $ vi /etc/nginx/conf.d/myangular.conf

Then insert the code below into myangular.conf.
```
upstream myangular {
  server localhost:4200;
}
server {
  listen 80;
  listen [::]:80;
  server_name mydomain.com www.mydomain.com *.mydomain.com server_ip;
  charset UTF-8;
  access_log /www/mysite/mynodeapp/log/access.log;
  error_log /www/mysite/mynodeapp/log/error.log;
  location / { 
    proxy_pass http://myangular;
  }
}
```

Check if any syntax error of nginx.
> $ nginx -t

Reload and restart nginx.
> $ systemctl reload nginx
> $ systemctl restart nginx

To check the status and log of nginx:
> $ journalctl -u nginx
> $ systemctl status nginx

Finally, we can open the site "myangular.mydomain.com" on a browser to verify the result.


## References
- [See more topics in my website](http://www.tzuchikao.com/en/notes/)
- [Switching between node versions during development](https://blog.logrocket.com/switching-between-node-versions-during-development/)
- [Node version manager](https://github.com/nvm-sh/nvm)
- [Express 4.x API](https://expressjs.com/zh-tw/4x/api.html)
- [Mean app with angular 2 and the angular cli](https://scotch.io/tutorials/mean-app-with-angular-2-and-the-angular-cli)
- [Configuring nginx Plus as a web server](https://docs.nginx.com/nginx/admin-guide/web-server/web-server/)


