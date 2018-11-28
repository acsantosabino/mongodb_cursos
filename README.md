# mongodb_cursos
Cursos de mongo da [MongoDB University](https://university.mongodb.com/)

## M103: Basic Cluster Administration

Diretório M103

* Executar vagrant env

``` bash
cd ./M103/m103-vagrant-env/ | vagrant ssh
```

### Chapter 1: The Mongod

#### Lab1 - Launching Mongod

``` bash
mongod --port 27000 --dbpath /data/db --bind_ip localhost,192.168.103.100 --auth --fork --logpath errors.log
```

#### Lab2 - Configuration File

* Arquivo de configuração em:

```bash
mongod --config /shared/lesson1Lab2.conf
```

#### Lab3 - Change the Default DB Path

```bash
mongod --config /shared/lesson1Lab3.conf
```

#### Lab4 - Logging to a Different Facility

```bash
mongod --config /shared/lesson1Lab4.conf
```

#### Lab5 - Creating First Application User

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

#### Lab6 - Importing a Dataset

```bash
mongoimport --port 27000 -u "m103-application-user" -p "m103-application-pass" --authenticationDatabase "admin" --db=applicationData --collection=products
```