# How to install MongoDB on CentOS and initialize by a Javascript file

## Version
- MongoDB 4.2.0
- CentOS 7


## Installation
Create a .repo file so that we can install MongoDB directly using yum.
> vi /etc/yum.repos.d/mongodb-org-4.2.repo

Insert the code below into mongodb-org-4.2.repo and save the file.
```
  [mongodb-org-4.2]
  name=MongoDB Repository
  baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/
  gpgcheck=1
  enabled=1
  gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
```

Check the packages of RPM Repository and make sure "repo id: !mongodb-org-4.2/x, repo name: MongoDB Repository" is in the list.
> sudo yum repolist
> or
> sudo yum repolist all

Install the latest stable version or specific version of MongoDB.
> sudo yum install -y mongodb-org
> or
> sudo yum install -y mongodb-org-4.2.0 mongodb-org-server-4.2.0 mongodb-org-shell-4.2.0 mongodb-org-mongos-4.2.0 mongodb-org-tools-4.2.0

Now we can start MongoDB service and check its status.
> sudo service mongod start
> sudo service mongod status


## Configuration
Assign the database path to MongoDB.
> sudo mongod -dbpath ~/data/db


## Execution and initialization
Create a javascript file to initialize the database.
> vi ~/data/db/testscript.js

Depends on the naming we need, modify the javascript code below, and then insert into testscript.js and save the file. It's used to create databases, users, user permissions, collections and documents.
```
var conn=null;
var db=null;
var dbadmin=null;
try {
  conn = new Mongo("127.0.0.1:27017");
  try {
    dbadmin = conn.getDB("admin");
    dbadmin.auth( "root", "123456" );
    db = dbadmin.getSiblingDB('mydatabase');
    try {
      db.createUser({
        "user" : "mydbuser",
        "pwd" : "123456789",
        "roles" : [
          { "db" : "mydatabase", "role": "readWrite" },
          { "db" : "mydatabase", "role": "dbAdmin" },
          { "db" : "mydatabase", "role": "dbOwner" },
          { "db" : "mydatabase", "role": "userAdmin" } ] });
    } catch (e) { print ('<< Create user FAIL >>'+e); }
    try {
      db.createCollection("firstCollection", { autoIndexId: true } );
      db.firstCollection.insertMany([
        { "field1": 1.00, "field2":{"item1":"value1", "item2":"value2"} },
        { "field1": 2.00, "field2":{"item1":"value3", "item2":"value4"} },
        { "field1": 3.00, "field2":{"item1":"value5", "item2":"value6"} } ]);
    } catch (e) {print ('<< Create or insertMany in firstCollection FAIL >>'+e); }
  } catch (e) { print ('<< Auth or create database FAIL >>'+e); }
} catch (e) { print ('<< Connection FAIL >>'+e); }
```

Enter the MongoDB shell.
> sudo mongo

In MongoDB shell, execute the initial file, check the result, and then exit the shell.
> \> load("/root/data/db/testscript.js")
> \> show dbs
> \> use mydatabase
> \> show users
> \> db.firstCollection.find()
> \> exit


## Notice!
Anytime, before reboot the os, we need to smoothly stop mongo service.
> sudo mongod --shutdown

Otherwise, mongod will be locked and causes the error:
  _mongod forked process: 893_
  _ERROR: child process failed, exited with error number 100_
Then we need to repair and restart mongo service.
> sudo mongod --repair --dbpath=/data/db --port=27017 --logpath=/usr/log/mongodb.log --repairpath /tmp/mongodb
> sudo systemctl restart mongod


## References
- [See more topics in my website.](http://www.tzuchikao.com/en/notes/)
- [Install MongoDB Community Edition on Red Hat or CentOS](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-red-hat/)
- [Manage mongod Processes](https://docs.mongodb.com/manual/tutorial/manage-mongodb-processes/)




