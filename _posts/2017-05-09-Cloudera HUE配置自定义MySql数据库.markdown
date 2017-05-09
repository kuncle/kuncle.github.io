---
layout: post
title:  "Cloudera HUE配置自定义MySql数据库"
date:   2017-05-09 15:30:00
categories: Cloudera
tags: Cloudera
---
ref：http://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_mysql.html#cmig_topic_5_5

###### In the Cloudera Manager Admin Console, go to the Hue service status page.
1. Select Actions > Stop. Confirm you want to stop the service by clicking Stop.
2. Select Actions > Dump Database. Confirm you want to dump the database by clicking Dump Database.
``` shell
Note the host to which the dump was written under Step in the Dump Database Command window. You can also find it by selecting Commands > Recent Commands > Dump Database.
Open a terminal window for the host and go to the dump file in /tmp/hue_database_dump.json.
Remove all JSON objects with useradmin.userprofile in the model field, for example:
{
"pk": 14,
"model": "useradmin.userprofile",
"fields":
{ "creation_method": "EXTERNAL", "user": 14, "home_directory": "/user/tuser2" }
},
Set strict mode in /etc/my.cnf and restart MySQL:
[mysqld]
sql_mode=STRICT_ALL_TABLES
Create a new database and grant privileges to a Hue user to manage this database. For example:
mysql> create database hue;
Query OK, 1 row affected (0.01 sec)
mysql> grant all on hue.* to 'hue'@'localhost' identified by 'secretpassword';
Query OK, 0 rows affected (0.00 sec)
```
3. In the Cloudera Manager Admin Console, click the Hue service. Click the Configuration tab.
4. Select Scope > All.
5. Select Category > Database.
``` shell
Specify the settings for Hue Database Type, Hue Database Hostname, Hue Database Port, Hue Database Username, Hue Database Password, and Hue Database Name. For example, for a MySQL database on the local host, you might use the following values:
Hue Database Type = mysql
Hue Database Hostname = host
Hue Database Port = 3306
Hue Database Username = hue
Hue Database Password = secretpassword
Hue Database Name = hue
```
6. Optionally restore the Hue data to the new database: Select Actions > Synchronize Database.
``` shell
Determine the foreign key ID.
$ mysql -uhue -psecretpassword
mysql > SHOW CREATE TABLE auth_permission;
(InnoDB only) Drop the foreign key that you retrieved in the previous step.
mysql > ALTER TABLE auth_permission DROP FOREIGN KEY content_type_id_refs_id_XXXXXX;
Delete the rows in the django_content_type table.
mysql > DELETE FROM hue.django_content_type;
```
7. In Hue service instance page, click Actions > Load Database. Confirm you want to load the database by clicking Load Database.
``` shell
(InnoDB only) Add back the foreign key.
mysql > ALTER TABLE auth_permission ADD FOREIGN KEY (content_type_id) REFERENCES django_content_type (id);
```
8. Start the Hue service.
