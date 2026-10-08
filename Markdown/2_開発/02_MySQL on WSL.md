[![hackmd-github-sync-badge](https://hackmd.io/YIWaeRG_SriNtTCLaNXx-w/badge)](https://hackmd.io/@hayat01sh1da/mysql-on-wsl)

<img src="https://hackmd.io/_uploads/HJQXYYBuA.png" alt="MySQL" />

## 1. Environment

Ubuntu 20.04.5 LTS

## 2. Install MySQL

### 2-1. Install MySQL Client

```bash
$ sudo apt install mysql-client
```

### 2-2. Install MySQL Server

```bash
$ sudo apt install mysql-server
```

### 2-3. Check Version

Check if MySQL is installed.

```bash
$ mysql --version
mysql  Ver 14.14 Distrib 5.7.27, for Linux (x86_64) using  EditLine wrapper
```

### 2-4. Check Behaviour

#### 2-4-1. Start

```
$ sudo /etc/init.d/mysql start
```

#### 2-4-2. Stop

```
$ sudo /etc/init.d/mysql stop
```

#### 2-4-3. Restart

```
$ sudo /etc/init.d/mysql restart
```

## 3. Reset Root Password

### 3-1. Prepare for Connecting to MySQL without Authentication

```bash
$ sudo mysqld_safe --skip-grant-tables &
[1] 5676
```

\* `--skip-grant-tables &` allows anyone to log in MySQL anywhere and anytime.  
It is highly dangerous, so it should be used only in the initial settings.
Please refer to [Official Document](https://dev.mysql.com/doc/mysql-security-excerpt/5.7/en/grant-tables.html#grant-tables-user-db) for the details.

### 3-2. Connecting to MySQL without Password as Root User

Log in as the root user with `-u`.

```bash
$ sudo mysql -u root
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.7.27-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type "help;" or "\h" for help. Type "\c" to clear the current input statement.

mysql>
```
### 3-3. Password Setting

```sql
mysql> UPDATE mysql.user SET authentication_string = "", password_expired = "N" WHERE User = "root";
mysql> UPDATE mysql.user SET plugin="mysql_native_password" WHERE User="root";
mysql> FLUSH PRIVILEGES;
exit
```

Please see the explanation of each SQL shown below.

#### 3-3-1. UPDATE mysql.user SET authentication_string = "", password_expired = "N" WHERE User = "root";

`DB_NAME.TABLE_NAME` finds the DB and table to update.  
Set the password in the `authentication_string` column and `N` in `password_expired` where the user is `root`.

#### 3-3-2. UPDATE mysql.user SET plugin = "mysql_native_password" WHERE User = "root;

Set `mysql_native_password` in `authentication_string` column for authentication plugin in the account where the user is `root`.

#### 3-3-3. FLUSH PRIVILEGES;

Reload on Grant Tables to reflect all changes.

## 4. Connect to MySQL

### 4-1. Prepare for Connecting to MySQL with Authentication

```bash
$ sudo mysqld_safe
```

### 4-2. Connect to MySQL with Password

Log in as the root user with the password using `-u` and `p`.

```bash
$ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.7.27-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type "help;" or "\h" for help. Type "\c" to clear the current input statement.

mysql>
```

Now MySQL setup is done.

## 5. Notes

Refer to the `user` table in `mysql` database with the SQL below.

### 5-1. Show DB

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.01 sec)
```

### 5-2. Go into DB

```sql
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
```

### 5-3. Show Tables

```sql
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| engine_cost               |
| event                     |
| func                      |
| general_log               |
| gtid_executed             |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| server_cost               |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
31 rows in set (0.01 sec)
```

### 5-4. Get Record from Table

```sql
mysql> select * from user;
+-----------+------------------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+-----------------------+-------------------+----------------+
| Host      | User             | Select_priv | Insert_priv | Update_priv | Delete_priv | Create_priv | Drop_priv | Reload_priv | Shutdown_priv | Process_priv | File_priv | Grant_priv | References_priv | Index_priv | Alter_priv | Show_db_priv | Super_priv | Create_tmp_table_priv | Lock_tables_priv | Execute_priv | Repl_slave_priv | Repl_client_priv | Create_view_priv | Show_view_priv | Create_routine_priv | Alter_routine_priv | Create_user_priv | Event_priv | Trigger_priv | Create_tablespace_priv | ssl_type | ssl_cipher | x509_issuer | x509_subject | max_questions | max_updates | max_connections | max_user_connections | plugin                | authentication_string                     | password_expired | password_last_changed | password_lifetime | account_locked |
+-----------+------------------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+-----------------------+-------------------+----------------+
| localhost | root             | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | Y          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      |          |            |             |              |             0 |           0 |               0 |                    0 | auth_socket           |                                           | N                | 2019-11-08 00:00:00   |              NULL | N              |
| localhost | mysql.session    | N           | N           | N           | N           | N           | N         | N           | N             | N            | N         | N          | N               | N          | N          | N            | Y          | N                     | N                | N            | N               | N                | N                | N              | N                   | N                  | N                | N          | N            | N                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *HOGEHOGEHOGEHOGEHOGEHOGEHOGEHOGEHOGEHOGE | N                | 2019-11-08 00:00:00   |              NULL | Y              |
| localhost | mysql.sys        | N           | N           | N           | N           | N           | N         | N           | N             | N            | N         | N          | N               | N          | N          | N            | N          | N                     | N                | N            | N               | N                | N                | N              | N                   | N                  | N                | N          | N            | N                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *HOGEHOGEHOGEHOGEHOGEHOGEHOGEHOGEHOGEHOGE | N                | 2019-11-08 00:00:00   |              NULL | Y              |
| localhost | debian-sys-maint | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | Y          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *HOGEHOGEHOGEHOGEHOGEHOGEHOGEHOGEHOGEHOGE | N                | 2019-11-08 00:00:00   |              NULL | N              |
+-----------+------------------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+-----------------------+-------------------+----------------+
4 rows in set (0.00 sec)
```

\* [w3schools.com](https://www.w3schools.com/sql/default.asp) explains SQL usages in detail.
