---
layout: post
title:  "Centos7安装Oracle XE"
date:   2017-05-05 11:00:00
categories: Linux
tags: Linux
---
* 下载Oracle XE
http://www.oracle.com/technetwork/database/database-technologies/express-edition/downloads/index.html
* 解压缩Oracle XE安装程序
unzip oracle-xe-11.2.0-1.0.x86_64.rpm.zip
* 创建用户
``` shell
groupadd oinstall  //创建oracle数据库安装组
groupadd dba        //创建oracle数据库管理组
useradd -m -g oinstall -G dba oracle  //创建oracle用户
id oracle
uid=1001(oracle) gid=1001(oinstall) groups=1001(oinstall),502(dba)

passwd oracle      //为Oracle用户设置密码：
Changing password for user oracle.
New UNIX password:
BAD PASSWORD: it is based on a dictionary word
Retype new UNIX password:
passwd: all authentication tokens updated successfully.
```
* 建立安装目录
``` shell
chown -R oracle:oinstall /u01/app
chmod -R 775 /u01/app
```
* 开始安装
``` shell
cd Disk1
rpm -ivh oracle-xe-11.2.0-1.0.x86_64.rpm
```
* 运行配置oracle xe的命令
``` shell
/etc/init.d/oracle-xe configure
```
* 使用oracle用户修改bash_profile中环境变量
修改.bash_profile.在其中添加如下内容：
``` shell
TMP=/tmp
TMPDIR=$TMP
ORACLE_HOSTNAME=dbserver
ORACLE_UNQNAME=ORADB 
ORACLE_BASE=/u01/app/oracle 
ORACLE_HOME=$ORACLE_BASE/product/11.2.0/xe
ORACLE_SID=ORADB
PATH=$ORACLE_HOME/bin:$PATH 
LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib 
export TMP TMPDIR ORACLE_HOSTNAME ORACLE_UNQNAME ORACLE_BASE ORACLE_HOME ORACLE_SID PATH LD_LIBRARY_PATH
```
* 测试是否成功
``` shell
echo $ORACLE_BASE
sqlplus / as sysdba  #查看是否可以进入sql命令行
```
