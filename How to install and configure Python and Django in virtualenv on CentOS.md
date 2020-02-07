# How to install and configure Python and Django in virtualenv on CentOS
## Version
- Python 3.7.4
- pip 19.3.1
- virtulenv 16.7.6
- django 2.2.3
- CentOS 7


## Install and configure python specific version

Install all necessary packages for compiling Python3.
> $ yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gcc make libffi-devel

Download and decompress python.
> $ mkdir /home/download; cd /home/download

> $ wget https://www.python.org/ftp/python/3.7.4/Python-3.7.4.tgz

> $ tar -zxvf Python-3.7.4.tgz

Install python to specific path, this example assign the path to /usr/local/python3.
> $ cd /home/download/Python-3.7.4

> $ ./configure prefix=/usr/local/python3 --enable-shared

> $ make

> $ make install

Open .bashrc file for setting environmental variables. This file will be automatically read while centOS getting login shell.
> $ vi ~/.bashrc

Insert the variable settings below into .bashrc.
```
  alias python37='/usr/local/python3/bin/python3.7'
  alias pip37='/usr/local/python3/bin/pip3.7'
  LD_LIBRARY_PATH >> :/usr/local/python3/lib
```

The new settings of .bashrc will be applied after logout and relogin centOS. Without relogin action, we can apply it immediately by the commands below.
> $ source ~/.bashrc

or
> $ . ~/.bashrc

Check the settings of environmental variables.
> $ echo $LD_LIBRARY_PATH

Check the installed versions.
> $ python37 --version

We used the alias "python37" to distinguish with the default "python". 
If we really need to use "python" for current installation, change the default python settings to "python2".
In the two files below, change "/usr/bin/python" to "/usr/bin/python2".
> $ vi /usr/bin/yum

> $ vi /usr/libexec/urlgrabber-ext-down


## Install and configure virtual environment
Check pip version and upgrade it.
> $ pip37 --version

> $ pip37 install --upgrade pip

Check virtualenv version, upgrade it or install if not exist.
> $ pip37 show virtualenv
> $ pip37 install --upgrade virtualenv

or
> $ pip37 install virtualenv

Assign the python interpreter to use.
> $ virtualenv -p /usr/local/python3/bin/python3.7 

Create a virtual environment.
> $ virtualenv /www/mysite/myenv

To actative the virtual environment:
> $ source myenv/bin/activate

To exit current virtual environment:
> $ deactivate


## Install and configure django
Active the virtual environment and then install django.
> $ source myenv/bin/activate

> $ pip37 install django==2.2.3

In the same way, install other necessary packages after the virtual environment is active.
> $ pip37 install <package name>==<package version>

Create a new project or put the source code folder under /www/mysite. 
> $ django-admin.py startproject myproject

Migrate with database with some default Django tables.
> $ python37 /www/mysite/myproject/manage.py migrate

Generate the static files.
> $ python37 /www/mysite/myproject/manage.py collectstatic

Finally, we can launch the Django web service to verify the project.
> $ python37 /www/mysite/myproject/manage.py runserver

or
> $ python37 /www/mysite/myproject/manage.py runserver server_domain_or_IP:8080


## References
- [See more topics in my website](http://www.tzuchikao.com/en/notes/)
- [Common Python tools: using virtualenv, installing with pip, and managing packages](https://www.digitalocean.com/community/tutorials/common-python-tools-using-virtualenv-installing-with-pip-and-managing-packages)
- [Deploy django with virtualenv on CentOS 7](https://devops.ionos.com/tutorials/deploy-django-with-virtualenv-on-centos-7/)
- [How to install the django web framework on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-the-django-web-framework-on-centos-7)
- [On CentOS 7, install Python3.7](https://segmentfault.com/a/1190000015628625)


