# How to serve Django app with uWSGI and NGINX on CentOS
## Version
- Python 3.7.4
- pip 19.3.1
- virtulenv 16.7.6
- Django 2.2.3
- uWSGI 2.0.18
- NGINX 1.17.5
- CentOS 7

## Confirm NGINX is installed

Check the version of NGINX and the service status.
> $ nginx -v

> $ systemctl status nginx

Check the user named "nginx" in the group "nginx" exists.
> $ id nginx

or
> $ grep nginx /etc/passwd

or
> $ getent passwd nginx

The installation of NGINX please reference [here](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/).


## Django project skeleton
To understand more easily, we list the django project files first.
```
[myproject]/
├── [app]/
│   ├── __init__.py
│   ├── apps.py
│   ├── urls.py
│   └── views.py
├── [conf]/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── [static]/
└── manage.py
```

Also, in the example here, we have django project folder in "/www/mysite/myproject" and virtualenv working folder in "/www/mysite/myenv".

The installation of Python, pip, django and virtualenv please reference [here](http://www.tzuchikao.com/en/notes/essay/5dbf622be97675ab3645185d).


## Install and configure uWSGI

Find python and pip installed path for later setting. In the example here, we have python in "/usr/local/python3/bin/python3.7" and pip in "/usr/local/python3/bin/pip3.7".
> $ which python

> $ which pip


Active the virtual environment and then install uWSGI, and then uWSGI will be installed in "/www/mysite/myenv/bin/uwsgi".
> $ source /www/mysite/myenv/bin/activate

> $ pip install uwsgi==2.0.18


Verify the installed version of uWSGI.
> $ pip show uwsgi


Now we can start uWSGI with simplest options.
> $ /www/mysite/myenv/bin/uwsgi --http-socket server_domain_or_IP:8080 --wsgi-file /www/mysite/myproject/conf/wsgi.py

or
> $ mkdir /www/mysite/myenv/log

> $ /www/mysite/myenv/bin/uwsgi --http-socket server_domain_or_IP:8080 --wsgi-file /www/mysite/myproject/conf/wsgi.py --pidfile /www/mysite/myenv/log/uwsgi-master.pid


To stop uWSGI, use the commands below.
> $ pkill -f /www/mysite/myenv/bin/uwsgi -9

or
> $ /www/mysite/myenv/bin/uwsgi --stop /www/mysite/myenv/log/uwsgi-master.pid


For saving the execution options, we need to create a .ini file.
> $ vi /www/mysite/myenv/bin/uwsgi.ini

Then insert the settings below into uwsgi.ini.
```
[uwsgi]
http = server_domain_or_IP:8080
socket = /www/mysite/myuwsgi.sock
chown-socket = nginx:nginx
wsgi-file = /www/mysite/myproject/conf/wsgi.py
uid = nginx
gid = nginx
master = true
pidfile = /www/mysite/myenv/log/uwsgi-master.pid
virtualenv = /www/mysite/myenv
pythonpath = /www/mysite/myenv/bin
enable-threads = true
home = /www/mysite/myenv
chdir = /www/mysite/myproject/conf
module = django
static-map = /static=/www/mysite/myproject/static
env = DJANGO_SETTINGS_MODULE=conf.settings
vacuum = true
```

Now we can start uWSGI by the .ini file.
> $ /www/mysite/myenv/bin/uwsgi --ini /www/mysite/myenv/bin/uwsgi.ini


If there shows an error of conf.settings not found, we can add the statement below into the file "/www/mysite/myproject/conf/wsgi.py".
```
sys.path.append('/www/mysite/myproject')
```

## Make uWSGI as a system control service

Create a .service file for saving the directives of uWSGI execution.
> $ vi /etc/systemd/system/uwsgi.service

Then insert the directives below into uwsgi.service. (Requires systemd version 211 or newer)
```
[Unit]
Description=uWSGI Emperor for django app
After=syslog.target
[Service]
ExecStart=/www/mysite/myenv/bin/uwsgi --ini /www/mysite/myenv/bin/uwsgi.ini
RuntimeDirectory=uwsgi
Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all
[Install]
WantedBy=multi-user.target
```

Now we can use uWSGI as a system control service by the following commands
> $ systemctl status uwsgi

> $ systemctl start uwsgi

> $ systemctl stop uwsgi

> $ systemctl restart uwsgi

> $ systemctl enable uwsgi

> $ systemctl disable uwsgi

To reload uwsgi.service:
> $ systemctl daemon-reload

To check logs:
> $ journalctl -u uwsgi


## Create the conf file for nginx

Create a .conf file for nginx.
> $ vi /etc/nginx/conf.d/mydjango.conf

Then insert the code below into mydjango.conf.
```
upstream mydjango {
  server unix:/www/mysite/myuwsgi.sock;
}
server {
  listen 80;
  server_name mydjango mydjango.mydomain.com;
  charset UTF-8;
  access_log /www/mysite/myenv/log/nginx_access.log;
  error_log /www/mysite/myenv/log/nginx_error.log;
  location /static/ {
    alias /www/mysite/myproject/static/;
  }
  location / {
    include uwsgi_params;
    uwsgi_pass mydjango;
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

Finally, we can open the site "mydjango.mydomain.com" on a browser to verify the result.


## Notice!
The path of the static folder which is wrote in  mydjango.conf and settings.py should direct to the same path, otherwise nginx may not find the .js and .css files of the static folder.

In the file: /etc/nginx/conf.d/mydjango.conf
```
  location /static/ {
    alias /www/mysite/myproject/static/;
  }
```

In the file: /www/mysite/myproject/conf/settings.py
```
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```


## References
- [See more topics in my website](http://www.tzuchikao.com/en/notes/)
- [Quickstart for Python/WSGI applications](https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html)
- [uWSGI Options](https://uwsgi-docs.readthedocs.io/en/latest/Options.html)
- [Systemd](https://uwsgi-docs.readthedocs.io/en/latest/Systemd.html)
- [Configuring nginx Plus as a web server](https://docs.nginx.com/nginx/admin-guide/web-server/web-server/)


