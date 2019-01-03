---
layout: single
title:  "MySQL Master-Slave replication with a Docker"
toc: true
toc_label: "On the page"
categories: linux
tags: docker mysql linux
header:
  teaser: /assets/images/mysql-replication-teaser.jpg
  og_image: /assets/images/mysql-replication-teaser.jpg
---

In this article was described a simple configuration of MySQL Master-Slave replication with using Docker containers. 

## Prepare 

Let's begin. Firstly we need to write both basic configurations (in file `10-mysqld.cnf`): 
```
# The MySQL Server configuration file. 
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html 
[mysqld]
pid-file = /var/run/mysqld/mysqld.pid 
socket = /var/run/mysqld/mysqld.sock 
datadir = /var/lib/mysql symbolic-links = 0 
```

After that create catalogs to store master and slave configures. I.e. `master-config` and `slave-config` in a root home directory. And copy the file `10-mysqld.cnf` into the dirs. 

## Configure master

Next we writing master configurations that stored in `master-config\60-enable-replication.cnf` (the number `60-` is used to define a order to load config files, the name of file is not important): 
```
[mysqld] 
server-id = 1 # the number of server, that be unical 
log-bin = mysql-bin # the name of binary log
binlog_do_db = test_db # the name of replication database
```

### Start master server

Run docker master container that based of official MySQL image and exposed port 33060:
```
docker run -d --rm --name mysql-master \
    -p 33060:3306 \
    -e MYSQL_ROOT_PASSWORD=root_secret -e MYSQL_DATABASE=test_db \
    -v /root/db_master:/var/lib/mysql \
    -v /root/master-config:/etc/mysql/mysql.conf.d \
    mysql
```

We can map the existing data that stored in `/root/db_master`  or create new database and  keep db files into this dir.

Now enter the container from command prompt and connect to created database:
```
docker exec -ti mysql-master /bin/bash
root@c337863c7d3e:/# mysql -proot_secret
mysql>
```

### Create replica user

Create new replication user `replica` with grants `REPLICATION SLAVE`in mysql console:
```
CREATE USER 'replica'@'%' IDENTIFIED BY 'replica_strong_password';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';
FLUSH PRIVILEGES;
```

### Dump database

Dump selected database and copy it to the slave. Before we should block any user connection to modify data:
```
FLUSH TABLES WITH READ LOCK;
SET GLOBAL read_only = ON;
```

Run `mysqldump` command to create dump database. I run it on my host computer that having IP address `192.168.0.161`:
```
mysqldump -h 192.168.0.161 -P 33060 -u root -proot_secret test_db > test_db.sql
```

Before unlock database don't forget check last binary log position with command `SHOW MASTER STATUS`
```
mysql> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |      107 | test_db      |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

After that you can unlock database:
```
SET GLOBAL read_only = OFF;
UNLOCK TABLES;
```

## Configure slave

Create file `slave-config\60-enable-replication.cnf` and input next rows to it:
```
[mysqld]
server-id          = 2           # Slave server ID (next after Master)
relay_log          = mysql-relay 
log_bin            = mysql-bin  
binlog_do_db       = test_db     
read_only          = 1           # The Slave will work only in read-only mode
```

### Run slave server 

Run container that will be slave server and link it to the `mysql-master` container:
```
docker run -d --rm --name mysql-slave \
    --link mysql-master:db \
    -p 33061:3306 \
    -e MYSQL_ROOT_PASSWORD=root_secret -e MYSQL_DATABASE=test_db \
    -v /root/db_slave:/var/lib/mysql \
    -v /root/slave-config:/etc/mysql/mysql.conf.d \
    mysql
```

Instead of linking containers you can use the host IP and exposed port to the master container (33060).

### Deploy master dump

Deploy the created dump to the slave server:
```
mysql -h 192.168.0.161 -P 33061 -u root -proot_secret test_db < test_db.sql
```

Go into the container of the slave server:
```
docker exec -ti mysql-slave /bin/bash
root@53378642dd3a:/# mysql -proot_secret
mysql>
```

And run the command [CHANGE MASTER TO][change_to] to write a connection to the master in MySQL console with information that we has input early on the master:
```
CHANGE MASTER TO MASTER_HOST='db', MASTER_USER='replica', MASTER_PASSWORD='replica_strong_password',
MASTER_LOG_FILE = 'mysql-bin.000001', MASTER_LOG_POS = 107;
```

And finally start replication:
```
START SLAVE;
```

To check the status of replication use next command:
```
mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: db
                  Master_User: replica
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 4133
               Relay_Log_File: mysql-relay.000003
                Relay_Log_Pos: 4346
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 0
                   Last_Error:
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 4133
              Relay_Log_Space: 4715
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 0
               Last_SQL_Error:
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 1
                  Master_UUID: 23b67d70-fab5-11e7-ac09-0242ac11000f
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp:
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set:
            Executed_Gtid_Set:
                Auto_Position: 0
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
1 row in set (0.00 sec)
```