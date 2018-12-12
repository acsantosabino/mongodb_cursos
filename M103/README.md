# M103: Basic Cluster Administration

## Chapter 1: The Mongod

### Lab1 - Launching Mongod

``` bash
mongod --port 27000 --dbpath /data/db --bind_ip localhost,192.168.103.100 --auth --fork --logpath errors.log
```

### Lab2 - Configuration File

* Arquivo de configuração em:

```bash
mongod --config /shared/lesson1Lab2.conf
```

### Lab3 - Change the Default DB Path

```bash
mongod --config /shared/lesson1Lab3.conf
```

### Lab4 - Logging to a Different Facility

```bash
mongod --config /shared/lesson1Lab4.conf
```

### Lab5 - Creating First Application User

```bash
mongo admin --host localhost:27000 -u m103-admin -p m103-pass
db.createUser({
    user: "m103-application-user",
    pwd: "m103-application-pass",
    roles: [
      {role: "readWrite", db: "applicationData"}
    ]
})
```

### Lab6 - Importing a Dataset

```bash
mongoimport --port 27000 -u "m103-application-user" -p "m103-application-pass" --authenticationDatabase "admin" --db=applicationData --collection=products
```

## Chapter 2: Replication

### Lab1 - Initiate a Replica Set Locally

```bash
# Cria diretórios dos dbs e inicia cada um
mkdir -p /var/mongodb/db/{1,2,3}
mongod -f /shared/mongod-repl-1.conf
mongod -f /shared/mongod-repl-2.conf
mongod -f /shared/mongod-repl-3.conf

# Conecta no primeiro para inicializar o ReplicatonSet e criar usuario admin
mongo admin --port 27001
rs.initiate()
use admin
db.createUser({
  user: "m103-admin",
  pwd: "m103-pass",
  roles: [
          {role: "root", db: "admin"}
        ]
})

# Conecta com usuario para configurar o ReplicatonSet
mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.status()
rs.add("192.168.103.100:27002")
rs.add("192.168.103.100:27003")
```

### Lab2 - Remove and Re-Add a Node

```bash
mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.status()
rs.remove("192.168.103.100:27003")
rs.add("m103:27003")
```

### Lab3 - Writes with Failovers

```bash
mongo --host "m103:27003" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
use admin
db.shutdownServer()
mongo --host "m103-repl/192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
rs.status()
use testDatabase
db.new_data.insert({"m103": "very fun"},
 { writeConcern: { w: 3, wtimeout: 1000 }})
```

### Lab4 - Read Concern and Read Preferences

```bash
mongoimport --drop \
--host m103-repl/192.168.103.100:27002,192.168.103.100:27001,192.168.103.100:27003 \
-u "m103-admin" -p "m103-pass" --authenticationDatabase "admin" \
--db applicationData --collection products /dataset/products.json

mongo --host "192.168.103.100:27001" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
use admin
db.shutdownServer()
mongo --host "192.168.103.100:27003" -u "m103-admin" -p "m103-pass" --authenticationDatabase "admin"
use admin
db.shutdownServer()

use applicationData
rs.slaveOk()
db.products.find().readPref("primary") #Error
db.products.find().readPref("secondary")
db.products.find().readPref("primaryPreferred")
db.products.find().readPref("secondaryPreferred")
db.products.find().readPref("nearest")
```