---
title: Linux上使用定时任务Crontab
keyword: 
- Linux 
- centos 
- cron 
- crontab
date: 2017-06-10 18:40:20
tags:
- 定时器
- cron
categories: 工具
---

使用linux的定时器，在服务器上面定时执行某些任务，比如时间同步，比如Hexo的定时更新和生成

<!--more-->

# 安装
## 检查是否安装crontab
在命令行中输入如下命令，观察是否安装crontab
```shell
crontab -e
```
## 安装crontab
如果没有安装，使用yum来进行安装，在命令行中输入下令命令来安装
```shell
yum install vixie-cron crontabs 
```
> 其中vixie-cron软件包是cron的主程序，crontabs软件包是用来安装、卸装、或列举用来驱动 cron 守护进程的表格的程序。

## 启动crontab
cron 是linux的内置服务，但它不自动起来，可以用以下的方法启动、关闭这个服务：
```shell
service crond start //启动服务
service crond stop //关闭服务
service crond restart //重启服务
service crond reload //重新载入配置
service crond status //查看crontab服务状态
```

## 加入开机启动
命令行中输入如下命令，在crond加入开机启动
```shell
chkconfig --level 35 crond on
```

# 使用
## cron表达式
linux上cron表达式为5位，分别代表 

第一位|第二位|第三位|第四位|第五位
-----|------|-----|------|-----
分钟  |小时  |天   |月    |周

## 命令使用方法
1. 命令行中输入`crontab -e`
2. 在弹出的文本中输入 `* * * * * * command`
> 比如 `*/1 * * * * echo "hello world" > /log/corn_test` 


# 查看现有的cron配置
1. 在命令行中输入`crontab -l`
2. 直接查看文件`vim /var/spool/cron/root`,其中最后的`root`为当前的用户名

