---
layout: post
title:  "virtualbox使用hostonly连外网"
date:   2017-03-15 13:50:00
categories: Linux
tags: Linux
---
###### 背景
> 公司机器都在内网中，访问外网需要配置代理。所以现在的问题是，virtualbox使用hostonly方式网络配置，并且需要配置代理访问外网。

#### 配置hostonly
1. 在客户机的Network Connections中配置Local Area Connection的sharing,勾选Allow other network user to connect...
2. 查看VirtualBox Host-Only Network的ip（如192.168.56.1），此ip作为虚拟机的gateway，虚拟机ip设置为192.68.56.2
3. 修改虚拟机dns,和主机中dns（ipconfig /all查看）配置一致
#### 配置代理
配置代理有多种方式，此处为修改环境变量.bash_profile
在.bash_profile中加入：
``` shell
export http_proxy=”http://proxy_addr:port”
export https_proxy="http://proxy_addr:port"
export ftp_proxy="http://proxy_addr:port"
如果代理需要用户名和密码的话，这样设置：
export http_proxy=”http://username:password@proxy_addr:port”
```
