---
layout: post
title: Rails部署之道学习笔记
description: "Rails部署，capistrano，chef之类的工具的学习"
category : [rails, note]
---

## 缘起
----

昨天，手贱。买了本62块的《Rails部署之道》。思前想后，都觉得特别贵，尤其还是一本电子书。此时，就运用逆向思维，通过看完，将钱挣回来。

## 前提条件

Web服务器结构？Chef和Capistrano，第一第二部分

第一部分涉及内容： ruby最佳安装，Monit，安全措施，UFW，用户及公钥，Redis，Memcached

第二部分涉及内容：Capistrano和Chef的作用，Unicorn，Sidekiq后台作业(Resque的竞争对手)。

chef自动配置，Capistrano执行部署过程以及相关的交互。

## chef

chef，Opscode，DSL - 可重复使用的配置服务器的所用的命令。运行在服务器集群枢纽。Chef Solo - 单机模式，Knife和Knife Solo。

## cap 

cap 是部署软件的通用解决方案，最近学了两招: 

* cap staging deploy:setup - 设置远程服务器的目录版本。
* cap production deploy:[restart|stop|start] - 部署脚本中设置的启动rails server的任务

```
cap staging [deploy|production]
cap production deploy:rollback   # 回滚到之前的一个版本
```

如果只是修改了一两个问题，没必要全部重新部署一下，所以需要使用一个特殊的方法: 

```sh
cap staging deploy:upload FILES=app/views/social/index.html.erb
cap staging deploy:restart
```

这样速度会快很多，测试部署时使用的时间也会很快。

理解: cap部署脚本其实是使用了ssh工具，将代码上传克隆并部署。

## 后记

自买回来之后，一直没时间去看。
