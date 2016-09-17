
---
title: 使用Docker创建Elasticsearch服务
date: 2016-09-17 20:00:00
---

最近需要做的几个功能都是基于Elasticsearch，所以就花了些时间研究下。

## Elasticsearch是什么？

Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。


## linux
/etc/profile:此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行. 并从/etc/profile.d目录的配置文件中搜集shell的设置. 

/etc/bashrc:为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取. 

\~/.bash\_profile:每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该 文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件. 

\~/.bashrc:该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该 该文件被读取. 

另外,/etc/profile中设定的变量(全局)的可以作用于任何用户,而\~/.bashrc等中设定的变量(局部)只能继承/etc/profile中的变量,他们是"父子"关系. 

\~/.bash\_profile 是交互式、login 方式进入 bash 运行的 
\~/.bashrc 是交互式 non-login 方式进入 bash 运行的 
通常二者设置大致相同，所以通常前者会调用后者。 



## MAC OS

MAC OS上的shell执行顺序有点特殊：
/etc/profile -\>\~/.bashrc(or .zshrc) -\> /etc/bashrc
















