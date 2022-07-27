# Replication setup
Enter the __super user__ mode (`su -`).
## Aditional info
- source ip: `192.168.110.128`
- replica ip: `192.168.110.129`

## Source Config 
On the __Source server__ configure the config file.
Source config `sudo vim /etc/my.cnf.d/mysql-server.cnf`.
```text
# Custom
bind-address    = 192.168.110.128
server-id       = 1
log_bin         = /var/log/mysql/mysql-bin.log
binlog_do_db    = DB1
```
restart the server
```shell
sudo systemctl restart mysql
```
Enter the mysql shell
```shell
mysql -u root -p
```
Execute following actions, add replica user
```shell
mysql> CREATE USER 'replica_user'@'192.168.110.129' IDENTIFIED WITH mysql_native_password BY 'rte.uY694a8bNrW-MQ_mBz';
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT REPLICATION SLAVE ON *.* TO 'replica_user'@'192.168.110.129';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> UNLOCK TABLES;
Query OK, 0 rows affected (0.00 sec)

mysql> CREATE DATABASE DB1;
Query OK, 1 row affected (0.00 sec)
```
Verify status
```shell
mysql> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |     1052 | DB1          |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```
Remember __Position__ value (1052).

## Replica Config
On the __Replica server__ configure the config file.
Replica config `sudo vim /etc/my.cnf.d/mysql-server.cnf`.
```text
# Custom
server-id               = 2
log_bin                 = /var/log/mysql/mysql-bin.log
binlog_do_db            = DB1
relay-log               = /var/log/mysql/mysql-relay-bin.log
```
Enter the mysql shell
```shell
mysql -u root -p
```
Execute commands
```shell
mysql> CHANGE REPLICATION SOURCE TO
    -> SOURCE_HOST='192.168.110.128',
    -> SOURCE_USER='replica_user',
    -> SOURCE_PASSWORD='rte.uY694a8bNrW-MQ_mBz',
    -> SOURCE_LOG_FILE='mysql-bin.000001',
    -> SOURCE_LOG_POS=1052;
```
create to to replicate database
```shell
mysql> create database DB1;
```
Start replica
```shell
mysql> START REPLICA;
Query OK, 0 rows affected (0.00 sec)
```
Verify
```shell
mysql> SHOW REPLICA STATUS\G;
*************************** 1. row ***************************
             Replica_IO_State: Waiting for source to send event
                  Source_Host: 192.168.110.128
                  Source_User: replica_user
                  Source_Port: 3306
                Connect_Retry: 60
              Source_Log_File: mysql-bin.000001
          Read_Source_Log_Pos: 1623
               Relay_Log_File: mysql-relay-bin.000004
                Relay_Log_Pos: 324
        Relay_Source_Log_File: mysql-bin.000001
           Replica_IO_Running: Yes
          Replica_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Source_Log_Pos: 1623
              Relay_Log_Space: 533
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Source_SSL_Allowed: No
           Source_SSL_CA_File: 
           Source_SSL_CA_Path: 
              Source_SSL_Cert: 
            Source_SSL_Cipher: 
               Source_SSL_Key: 
        Seconds_Behind_Source: 0
Source_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Source_Server_Id: 1
                  Source_UUID: 1f46c456-0c20-11ed-8c71-000c294f88bc
             Source_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
    Replica_SQL_Running_State: Replica has read all relay log; waiting for more updates
           Source_Retry_Count: 86400
                  Source_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Source_SSL_Crl: 
           Source_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Source_TLS_Version: 
       Source_public_key_path: 
        Get_Source_public_key: 0
            Network_Namespace: 
1 row in set (0.00 sec)

ERROR: 
No query specified
```

## Test database replication
### Source
in mysql prompt of __Source__
```shell
mysql> USE DB1;
Database changed
mysql> CREATE TABLE example_table (
    -> example_column varchar(30)
    -> );
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO example_table VALUES
    -> ('This is the first row'),
    -> ('This is the second row'),
    ->     ('This is the third row');
Query OK, 3 rows affected (0.01 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> exit
```

### Replica
Verify if data is copied
```shell
mysql> use DB1
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------+
| Tables_in_DB1 |
+---------------+
| example_table |
+---------------+
1 row in set (0.00 sec)

mysql> select * from example_table
    -> ;
+------------------------+
| example_column         |
+------------------------+
| This is the first row  |
| This is the second row |
| This is the third row  |
+------------------------+
3 rows in set (0.00 sec)

mysql> exit  
```

## Usefull info
get users of DB
```shell
mysql> Select user from mysql.user; 
+------------------+
| user             |
+------------------+
| meadmin          |
| mysql.infoschema |
| mysql.session    |
| mysql.sys        |
| root             |
+------------------+
5 rows in set (0.00 sec)
```




