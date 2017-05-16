---
layout: post
title:  "Cloudera集群IP修改"
date:   2017-05-16 13:55:00
categories: Cloudera
tags: Cloudera
---
1, 首先在安装cloudera-manager的主机上，停止所有的cloudera管理进程
``` shell
service cloudera-scm-agent stop
service cloudera-scm-server-db stop (我用的外部数据库mysql,所以不需要这步)
service cloudera-scm-server stop
```

2, 查看postgresql的scm用户的密码 (我用的外部数据库mysql,所以不需要这步)
``` shell
grep password /etc/cloudera-scm-server/db.properties
```

3, 登录mysql / postgresql数据库
``` shell
mysql: mysql -uroot -p
postgresql: psql -h localhost -p 7432 -U scm
提示你输入密码,密码就是上面第二步的密码
```
4, 修改mysql / postgresql数据库中的数据（即主机的ip）
``` shell
查看ip
mysql: select host_id, host_identifier, name, ip_address from hosts;
postgresql: select host_id, host_identifier, name, ip_address from HOSTS;
修改各主机的ip（分别修改各主机的ip）
mysql: update HOSTS set ip_address = '新的ip' where host_id='1';
postgresql: update hosts set (ip_address) = ('新的ip') where host_id='1';
修改你需要修改的机器ip
```

5, 修改所有Hadoop集群机器中的cloudera-scm-agent的配置文件
``` shell
vi /etc/cloudera-scm-agent/config.ini
修改host为新的ip
```

6, 修改各主机的/etc/hosts文件，将现在的hostname与ip地址对应上

7, 重启服务
``` shell
service cloudera-scm-server-db start(我用的外部数据库mysql,所以不需要这步)
service cloudera-scm-server start
service cloudera-scm-agent start
```
