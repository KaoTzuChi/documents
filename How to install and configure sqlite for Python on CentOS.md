# How to install and configure sqlite for Python on CentOS
## Version
- Python 3.7.4
- wget 1.14
- make 3.82
- sqlite 3.30.1
- CentOS 7

## Installation
The installation of Python, wget and make please reference [here](http://www.tzuchikao.com/en/notes/essay/5dbf622be97675ab3645185d).

Download and decompress sqlite.
> cd /home/download
> wget http://www.sqlite.org/2019/sqlite-autoconf-3300100.tar.gz
> tar xvfz sqlite-autoconf-3300100.tar.gz

If the environment has existing sqlite, check the installed path. The result is normally "/usr/bin/sqlite3"
> which sqlite3

And then backup the existing sqlite.
> mv /usr/bin/sqlite3 /usr/bin/sqlite3_old

Install sqlite to specific path, this example assign the path to /usr/local/sqlite.
> cd /home/download/sqlite-autoconf-3300100
> ./configure prefix=/usr/local/sqlite
> make && make install

If the existing sqlite is backuped, make a symbolic link for the new installed sqlite.
> cd /usr/local/sqlite/bin/
> ln -s sqlite3 /usr/local/sqlite3


## Configuration
Open .bashrc file for setting environmental variables. This file will be automatically read while centOS getting login shell.
> vi ~/.bashrc

Insert the variable settings below into .bashrc.
```
  alias sqlite3='/usr/local/sqlite/bin/sqlite3'
  export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/sqlite/lib
  export LD_RUN_PATH=$LD_RUN_PATH:/usr/local/sqlite/lib
  export PATH=$PATH:/usr/local/sqlite/bin
```

The new settings of .bashrc will be applied after logout and relogin centOS. Without relogin action, we can apply it immediately by the commands below.
> source ~/.bashrc
> or
> . ~/.bashrc

Check the settings of environmental variables.
> echo $LD_LIBRARY_PATH
> echo $LD_RUN_PATH
> echo $PATH

Create a .conf file for sqlite to set the library path.
> vi /etc/ld.so.conf.d/sqlite.conf

Insert the path into sqlite.conf.
```
  /usr/local/sqlite/lib
```

Load dynamic libraries into cache memory, and then list and check the libraries about sqlite.
> ldconfig
> or
> ldconfig -p | grep sqlite


## Execution
Check installed version of sqlite. Additionally, we can just type "sqlite3" to enter sqlite sell.
> sqlite3 --version

Enter Python shell and check which version of sqlite is used.
> python
> \>\>\> import sqlite3
> \>\>\> sqlite3.sqlite_version
> or
> python
> \>\>\> from sqlite3 import dbapi2 as Database
> \>\>\> Database.sqlite_version_info


## References
- [See more topics in my website.](http://www.tzuchikao.com/en/notes/)
- [Upgrade To Latest SQLite3 on CentOS7](https://linuxhint.com/upgrade-to-latest-sqlite3-on-centos7/)
- [How To Compile SQLite](https://sqlite.org/howtocompile.html)
- [django ImproperlyConfigured: SQLite 3.8.3 or later is required](http://www.py3study.com/Article/details/id/2810.html)



