---
layout: post
title:  "centos7 安装和配置 rabbitmq"
date:   2017-05-27 11:06:05
categories: centos7 
tags: centos7 rabbitmq
---

* content
{:toc}

在 **centos 7 上安装和配置 rabbitmq。[官方文档](!http://www.rabbitmq.com/install-rpm.html)有更加详细的配置





## 安装 erlang

rabbitmq 需要有 erlang 的环境，如果使用 yum 安装，执行下面的命令
	
```
 su -c 'rpm -Uvh http://download.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm'
 su -c 'yum install foo’
 yum install erlang
```
执行完成后可以使用 erlang 进行检查是否安装成功。

## 安装 rabbitmq server

```
rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
yum install rabbitmq-server-3.6.10-1.noarch.rpm
```

## 启动
- 设置开机启动
```
chkconfig rabbitmq-server on
```
- 启动
```
service rabbitmq-server start
```
- 停止
```
service rabbitmq-server stop
```

## 安装 web 管理页面
```
rabbitmq-plugins enable rabbitmq_management
```
安装完成后就可以使用 http://ip:15672进行访问了，web 页面需要用户，所以下边就需要对用户进行配置

## 用户管理
- 用户列表
```
rabbitmqctl  list_users
```
- 新增用户
```
rabbitmqctl  add_user  Username  Password
```
- 删除用户
```
rabbitmqctl  delete_user  Username
```
- 修改密码
```
rabbitmqctl  change_password  Username  Newpassword
```

用户添加完成后还不能登录 web 页面，还需要给用户添加角色

## 用户角色

- 超级管理员(administrator)

       可登陆管理控制台(启用management plugin的情况下)，可查看所有的信息，并且可以对用户，策略(policy)进行操作。
        
- 监控者(monitoring)

       可登陆管理控制台(启用management plugin的情况下)，同时可以查看rabbitmq节点的相关信息(进程数，内存使用情况，磁盘使用情况等)

- 策略制定者(policymaker)

    可登陆管理控制台(启用management plugin的情况下), 同时可以对policy进行管理。但无法查看节点的相关信息。
    
- 普通管理者(management)

     仅可登陆管理控制台(启用management plugin的情况下)，无法看到节点信息，也无法对策略进行管理。

- 其他

    无法登陆管理控制台，通常就是普通的生产者和消费者。
    了解了这些后，就可以根据需要给不同的用户设置不同的角色，以便按需管理。
    
赋予用户角色
```
rabbitmqctl  set_user_tags  User  Tag
```
一个用户可以有多个角色，多个 tag 使用空格分开即可



