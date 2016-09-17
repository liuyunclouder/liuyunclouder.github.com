
---
title: 使用Docker创建Elasticsearch服务
date: 2016-09-17 20:00:00
---

最近需要做的几个功能都是基于Elasticsearch，但又不想污染系统环境，所以就花了些时间研究下如何使用Docker创建Elastcisearch服务。

首先先分别了解下Docker和Elasticssearch。


## Docker是什么？

Docker是一个开源工具，能将一个WEB应用封装在一个轻量级，便携且独立的容器里，然后可以运行在几乎任何服务环境下。
Docker的容器能使应用跑在任何服务器上并且表现一致。一个开发者在笔记本上建立的一个容器，能跑在很多环境下，如：测试环境，生产环境，虚拟机上，VPS，OpenStack集群，公用的电脑等等
Docker的一般使用在以下几点：
•	自动化打包和部署应用
•	创造一个轻量级的，私人的 PAAS 环境
•	自动化测试和连续的 整合/部署
•	部署WEB应用，数据库和后端服务


所以，Docker是一个系统级兼容的容器，它采用Linux Container技术构建一个虚拟环境，用户可以在这个环境下安装各种应用来提供服务，并且这个环境可以随时创建或销毁，不会影响宿主环境


## Elasticsearch是什么？

Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

不过，Elasticsearch不仅仅是Lucene和全文搜索，我们还能这样去描述它：
•	分布式的实时文件存储，每个字段都被索引并可被搜索
•	分布式的实时分析搜索引擎
•	可以扩展到上百台服务器，处理PB级结构化或非结构化数据

总之，ES是一个牛逼的搜索存储引擎

## 安装Docker

Mac上安装Docker很简单，基本跟着[官网][1]的引导就可以顺利安装了，完成后确保Docker已启动，如下图
![][image-1]

## 创建Docker 镜像
Elasticsearch官方在Docker Hub上已经有提供镜像，如果没有额外需求，执行下面这个命令就可以直接使用Elasticsearch官方提供的镜像：

	docker run -d -p 9200:9200 --name="es" elasticsearch:2.3.5

但我还想额外装一个Elasticsearch的插件，方便调适，所以就自己做了一个镜像，[ Dockerfile ]()

	FROM elasticsearch:2.3.5
	
	RUN /usr/share/elasticsearch/bin/plugin install mobz/elasticsearch-head
	
	EXPOSE 9200

进入Dockerfile所在的文件夹，执行以下命令：

	docker build --tag=es_ezio:2.3.5 .

然后执行

	docker ps

就能看到刚才创建的镜像了
![][image-2]

## 启动容器及服务

上一步我们只是制作了一个Docker镜像，还没有创建Docker容器。关于Docker中镜像和容器的关系，可以类比为操作系统中的程序和进程，或者面向对象语言中的Class和Instance。我们必须从镜像创建出容器才能运行我们的服务（也就是Elasticsearch服务）。

第一次创建Docker容器，执行以下命令：

	docker run -d -p 9200:9200 --name="es_ezio" es_ezio:2.3.5

Elasticsearch的默认端口是9200，我们把宿主环境9200映射到Docker容器中的9200端口，这样我们就可以直接访问宿主环境的9200端口就可以访问到Docker容器中的Elasticsearch服务了，同时我们把这个容器命名为es\_ezio。

如果一切顺利，访问 [http://127.0.0.1:9200/\_plugin/head/][3]
![][image-3]

这样，我们就完成了yo

## MAC OS


















[1]:	https://docs.docker.com/docker-for-mac/ "https://docs.docker.com/docker-for-mac/"
[3]:	http://127.0.0.1:9200/_plugin/head/

[image-1]:	http://d.pr/i/13gIp+
[image-2]:	http://d.pr/i/1k6a8+
[image-3]:	http://d.pr/i/n8i8+