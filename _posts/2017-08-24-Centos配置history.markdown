---
layout: post
title:  "Centos配置history"
date:   2017-08-24 10:00:00
categories: Linux
tags: Linux
---
history显示的信息有局限性，默认保存最近的1000条命令，从历史信息中只能看到某个命令的执行有可能导致系统出了问题，如果想定位到是哪个用户在哪个时间在哪执行的命令，需要自己配置。   
在/etc/profile中加入以下脚本
``` shell
#history  
USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`  
HISTDIR=/usr/share/.history  
if [ -z $USER_IP ]  
then  
USER_IP=`hostname`  
fi  
if [ ! -d $HISTDIR ]  
then  
mkdir -p $HISTDIR  
chmod 777 $HISTDIR  
fi  
if [ ! -d $HISTDIR/${LOGNAME} ]  
then  
mkdir -p $HISTDIR/${LOGNAME}  
chmod 300 $HISTDIR/${LOGNAME}  
fi  
export HISTSIZE=4000  
DT=`date +%Y%m%d_%H%M%S`  
export HISTFILE="$HISTDIR/${LOGNAME}/${USER_IP}.history.$DT"  
export HISTTIMEFORMAT="[%Y.%m.%d %H:%M:%S]"  
chmod 600 $HISTDIR/${LOGNAME}/*.history* 2>/dev/null 
```
上面的脚本可以记录用户ip和时间，在/usr/share/.history下以用户名命名的目录下，历史记录文件名根据用户ip和时间命名。
