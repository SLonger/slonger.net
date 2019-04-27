---
title: "利用 ssh  反向代理外部访问内网服务"
date: 2019-04-27T00:00:00+08:00
lastmod: 2019-04-27T00:00:00+08:00
tags: ["preview", "Theme preview", "tag-3"]
categories: ["ssh"]

weight: 10
contentCopyright: MIT 
mathjax: true
autoCollapseToc: true

---

<!--more-->



 # 前言
* 场景
       需要在外部环境访问家里的电脑
* 准备环境
 1.  一个具有公网ip的A 服务器 vps  example 45.32.172.16 (  ubuntu 16.04(根据情况自己配置) 
 2.  需要能够被外部访问的内网B 机器（ubuntu 18.04)

# 配置
## 一.  ssh 设置 :B 机器上 配置ssh
 ssh 配置 反向代理的语法：( from man ssh)

  > -R [bind_address:]port:host:hostport   
  
example 
```
 $ ssh -NfR 7001:localhost:22 root(example)@45.32.172.16 -p 22 
```

> -R :  做反向ssh，将远程服务器的7001端口映射成连接本机(B)与该服务器的反向ssh的端口
> -N:只用于端口转发，并不执行远程shell
> -f:在执行命令前，进入后台处理 
> bind_address: 远程服务器（A) 由于ssh 本身需要使用到远程服务器 所以这里可以忽略 
> post: 远程端口 该端口映射到内网本地端口 
> host: 内网 B ip hostport: 内网 B 端口
> \-p:  ssh 默认端口是22 如果没有设置其他端口 可以不写 （-p 22)

执行效果：

-  B 机器 后台一直在运行 ssh 命令
- 45.32.172.16：7001 将会被映射成 内网b机器22 端口（设置成 22  直接到时免密登陆）
- 可以在 a 机器上看是否 在监听 7001 端口

> $netstat -anp | grep 7001 
>  tcp        0      0 0.0.0.0:7001           
> 	0.0.0.0:*               LISTEN      2646/sshd: root  tcp6       0      0 :::7001                 :::*                    LISTEN     
> 2646/sshd: root


----- 这里可以直接先用测试方式进行测试看是否配置成功-----

常见问题：
如果按照上面未配置成功 可能有以下问题：

1 . 外网服务器上的 SSH 没有配置对。为此你需要去外网服务器上修改  或者添加  一行 `/etc/ssh/sshd_config` 文件如下：
```
GatewayPorts yes
```
这个选项的意思是，SSH 隧道监听的服务的 IP 是对外开放的 0.0.0.0，而不是只对本机的 127.0.0.1。不开 GatewayPorts 的后果是不能通过 `45.32.172.16:7001` 访问，只能在外网服务器上通过 `127.0.0.1:7001` 服务到本地开发机的服务。
重启 sshd 服务
```
service sshd restart
```

2.A 服务器防火墙未开启 7001  需要设置。


------
## 二. autossh 配置
在b机器上配置的ssh跟服务断开时，自动重新连接 避免需要在机器上手动启动
这里由于autossh 直接进入后台执行命令 而不会有输入密码提示：导致无法连接
所以先设置下免密登陆.
在内网的机器B上面执行

    $ssh-copy-id  root@45.32.172.16
    $sudo apt-get install autossh
    $ autossh -M 2222 -NfR 7001:localhost:22 root@45.32.172.16 -p 22

> \-M: 另外再设置端口用于监听 ssh 连接情况

## 三 .测试方式：
	找一台可以远程 a 服务器的电脑执行：


> ssh -p 7001 username(localusername)@45.32.172.16
 
注意： 这里可能跟 你在b机器上 的username不一致。例如

你想在外部访问的是B 机器   long(用户名）则上面该命令是
> ssh -p 7001  long@45.32.172.16

而不是
> ssh -p 7001   root@45.32.172.16

否则会不断出现密码输入错误的提示

## 四 .简单代理( 太长 直接复制)

还有一种方式叫作动态转发，命令如下：

```
ssh -D 50000 root@45.32.127.32。
```

这种方式其实就是相当于socks代理，他会把本地的所有请求都转发到远程服务器上面，很实用哦，假如说你的那台服务器是在国外的话，你懂的！


**参考连接：**
1.   [ [利用ssh反向代理以及autossh实现从外网连接内网服务器(https://www.cnblogs.com/kwongtai/p/6903420.html)]  
2. [利用反向ssh从外网访问内网主机]    (https://blog.mythsman.com/2017/01/14/1/)
3. [# [调试利器-SSH隧道] (https://segmentfault.com/a/1190000011846777)]

OVER!!
