# Docker MySQL master-slave replication

MySQL 8.0 master-slave replication with using Docker.

Previous version based on MySQL 5.7 is available in [mysql5.7](https://github.com/vbabak/docker-mysql-master-slave/tree/mysql5.7) branch.

## Run

To run this examples you will need to start containers with "docker-compose"
and after starting setup replication. See commands inside ./build.sh.

#### Create 2 MySQL containers with master-slave row-based replication

```bash
./build.sh
```

#### Follow this step if unable to run above

Stop and remove networks

```bash
docker-compose down -v
```

Delete any old mysql data

```bash
rm -rf ./master/data/*
```

```bash
rm -rf ./slave/data/*
```

Build docker compose file

```bash
docker-compose build
```

Bringup the containers

```bash
docker-compose up -d
```

Get Docker process

```bash
docker ps --all
```

Run Master server with Root access

```bash
docker-compose exec mysql_master mysql -u root -p
```

Check server_id of Master

```mysql
SHOW variables LIKE 'server_id';
```

Create User for Replication with privileges

```mysql
CREATE USER 'slave'@'%' IDENTIFIED WITH mysql_native_password BY 'slave123@'; GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%'; FLUSH PRIVILEGES;
```

Get Master Status and File as MASTER_LOG_FILE, Position as MASTER_LOG_POS

```mysql
SHOW MASTER STATUS\G
```

Run Slave server with Root access

```bash
docker-compose exec mysql_slave mysql -u root -p
```

Check server_id of Slave, ensure Master and slave have different server_id

```mysql
SHOW variables LIKE 'server_id';
```

Update server_id

```mysql
SET GLOBAL server_id = 2;
```

Change Master and add MASTER_LOG_FILE and MASTER_LOG_POS of Master

```mysql
CHANGE MASTER TO MASTER_HOST='mysql_master',MASTER_USER='slave',MASTER_PASSWORD='slave123@',MASTER_LOG_FILE='binlog.000002',MASTER_LOG_POS=829; START SLAVE;
```

Get Slave Status

```mysql
SHOW SLAVE STATUS \G
```

Ensure Slave_IO_Running, Slave_SQL_Running has Yes status and Last_IO_Error is Clear

#### Make changes to master

```bash
docker exec mysql_master sh -c "export MYSQL_PWD=111; mysql -u root mydb -e 'create table code(code int); insert into code values (100), (200)'"
```

#### Read changes from slave

```bash
docker exec mysql_slave sh -c "export MYSQL_PWD=111; mysql -u root mydb -e 'select * from code \G'"
```

## Troubleshooting

#### Check Logs

```bash
docker-compose logs
```

#### Start containers in "normal" mode

> Go through "build.sh" and run command step-by-step.

#### Check running containers

```bash
docker-compose ps
```

#### Clean data dir

```bash
rm -rf ./master/data/*
rm -rf ./slave/data/*
```

#### Run command inside "mysql_master"

```bash
docker exec mysql_master sh -c 'mysql -u root -p111 -e "SHOW MASTER STATUS \G"'
```

#### Run command inside "mysql_slave"

```bash
docker exec mysql_slave sh -c 'mysql -u root -p111 -e "SHOW SLAVE STATUS \G"'
```

#### Enter into "mysql_master"

```bash
docker exec -it mysql_master bash
```

#### Enter into "mysql_slave"

```bash
docker exec -it mysql_slave bash
```
