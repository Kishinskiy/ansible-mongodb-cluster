# Mongodb cluster

[ Guide ](https://jagadeeshs.wordpress.com/2015/12/24/mongodb-cluster-setup/)

## MongoDB Cluster setup 
### Sharded Cluster Deployment

#### Introduction to Sharding

Sharding refers to the process of splitting data up across machines; the term partitioning is also sometimes used to describe this concept. By putting a subset of data on each machine, it becomes possible to store more data and handle more load without requiring larger or more powerful machines, just a larger quantity of less-powerful machines.

First, we need to install MongoDB in all three given Ubuntu servers

1) 192.168.1.103

2) 192.168.1.104

3) 192.168.1.105

#### Installation

1.Import the public key used by the package management system.

```bash
sudo apt-key adv --keyserver
hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10
```

2. Create a list file for MongoDB.

```bash
echo "deb http://repo.mongodb.org/apt/ubuntu "$(lsb_release-sc)"/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list
```

3.Reload local package database.
```bash
sudo apt-get update
```


4.Install MongoDB packages.
```bash
sudo apt-get install -y mongodb-org
```


 
Config Servers: (192.168.33.10:27019)
--------------------------------------
Config servers store the cluster’s metadata. This data contains a mapping of the cluster’s data set to the shards. The query router uses this metadata to target operations to specific shards.

Before Configuring Config server we need to create /data/configdb directory to specify dbpath and we need to give ownership for the path as mongodb.And we need to Generate a certificate file to specify keyFile path and copy the same for all nodes in a cluster.

Generating a KeyFile:
```bash
openssl rand -base64 741 > mongodb-keyfile
chmod 600 mongodb-keyfile
```

Now we need to configure config server by creating a file like
```editorconfig
# /etc/mongodb_config.conf

# mongodb configdb server config file
port = 27019
dbpath = /data/configdb
fork = true
configsvr = true
logpath = /var/log/mongodb/mongodb_config.log
logappend = yes
#keyFile = /home/developer/mongodb-keyfile
#auth = true
```


Query Routers: (192.168.1.103:27020)
------------------------------------

Query Routers are basically mongos instances, interface with client applications and direct operations to the appropriate shard. The query router processes and targets operations to shards and then returns results to the clients. A sharded cluster can contain more than one query router to divide the client request load. A client sends requests to one query router. Generally a sharded cluster have many query routers.

Now we need to configure mongos server by creating a file like
```editorconfig

# /etc/mongos.conf

# mongos config file
port = 27020
configdb = 192.168.1.103:27019
fork = true
logpath = /var/log/mongos.log
logappend = yes
#keyFile = /home/developer/mongodb-keyfile

```
Here we need to specify the config server name in configdb option.

Shard Servers: (192.168.1.104 , 192.168.1.105)
----------------------------------------------

Shards: Shards are used to store data. They provide high availability and data consistency. In production environment each shard is a separate replica set.

Now we need to configure shard servers by creating a file in the two shard servers like
```editorconfig

# /etc/mongodb_shard.conf

# mongod replicated sharded config file
port = 27018
dbpath = /data/db
fork = true
logpath = /var/log/mongodb.log
logappend = yes
shardsvr = true
#keyFile = /home/development/mongodb-keyfile
#auth = true
```

After creating all files we need to start all mongod and mongos instances:

1. Start Mongod Instance in config server (192.168.1.103)
```bash
sudo mongod -f /etc/mongodb_config.conf
```

2.Start Mongos Instance in Mongos server (192.168.1.103)
```bash
sudo mongos -f /etc/mongos.conf
```

3.Then Start Mongod Instance in both shard servers (192.168.1.104 and 192.168.1.105)
```bash
sudo mongod -f /etc/mongodb_shard.conf
```


Adding shards to the cluster

From a mongo shell, connect to the mongos instance. Issue a command using the following syntax:
```bash
sudo mongo 192.168.1.103:27020/admin
```


Add each shard to the cluster using the sh.addShard() method


>sh.addShard(“192.168.33.20:27018”)

>sh.addShard(“192.168.33.30:27018”)

ENABLE SHARDING FOR A DATABASE

Before we shard a collection, we must enable sharding for the collection’s database. Enabling sharding for a database does not redistribute data but make it possible to shard the collections in that database.

>sh.enableSharding(“dbname”)

ENABLE SHARDING FOR A COLLECTION

Before enabling sharding, we have to create an index on the key we want to shard by:

> db.users.ensureIndex({"username" : 1})

Now we’ll shard the collection by "username":

> sh.shardCollection("test.users", {"username" : 1})

We can able to see status of sharding by using

>sh.status()

Or

>sh.printShardingStatus()

Create an Administrative User with Unrestricted Access

>use admin

>
db.createUser({ user: “superuser”, pwd: “12345678”,roles: [ “root” ] })

After Creating Administrative user we need to enable key file in the configuration files by removing # symbol and then restart the instance.

Access Database through the command

```bash
$sudo mongo 192.168.33.10:27020/admin -u admin –password
```

