
---
title: Arsos — a lib search engine for coder
date: 2016-10-22 20:00:00
---

Arsos，本来是二战时期协和国执行的一项搜寻德国原子核实验地点行动的名称，由于这项行动，协和国才真正了解了德国的原子核计划的真实进展，并成功破坏了其重水的来源。

同时，Arsos也是一个为工程师量身定做的搜索库和组件的Mac APP。

### 由来

Arsos来源于某天看到公司的前端工程师在查找一个js库，步骤非常繁琐：先打开浏览器，输入网址，然后输入angular，得到一长串结果，然后还要根据版本号筛选，如果想要找到angular下的cookie组件，还得Command + F，接下来输入cookie …脑细胞已阵亡。

作为一个熟练使用Alfred、Popclip、BTT的效率小能手（check 我自制的[ Alfred workflow][1] 、[ Popclip extension ][2]），简直不能忍，于是就萌生了自己做一个工具来简化这个流程。

### 整体架构
![]()
服务端由两个Server和一个Database组成（缓存没太大必要，文件存储暂时没太大需求，暂时Droplr够用）：Data Server负责收集数据并按照Service Server中定义的model的格式把数据保存到DB中，Service Server目前主要扮演API网关的角色，为Mac APP提供API接口，另外还会兼职Web Site中的展示和数据源后台管理。

客户端由Mac APP和Web Site组成：Mac APP主要对外提供搜索和下载服务，Web Site对外提供产品介绍的同时对内提供数据源管理服务，方便管理员编辑数据。

### 功能介绍
作为一个Alfred重度患者，为了向Alfred致敬（是的，这不叫抄袭），UI设计和交互方式基本都是参照它来设计的，所以对用过Alfred的朋友来说是非常友好的，如果没用过Alfred也没关系，下面简单介绍下现有的功能。

#### 快速唤起
下载安装后（[下载地址][3]），运行Arsos，然后切换到其它应用，比如Atom或者sublime，按下快捷组合键：

Shift + 空格（自定义）

即可唤起Arsos进行搜索。

![][image-2]

#### 搜索
搜索功能使用非常简单，输入的关键词以空格分割，回车后就会返回所有模糊匹配成功的结果。

![][image-3]

#### 复制
在搜索结果列表中，使用上下箭头选中结果，回车即可复制选中结果到剪贴板中
![][image-4]

另外，如果回车的同时按住option键，即可复制带有标签的结果
![][image-5]


#### Alfred workflow
同时还给Alfred的重度患者提供一个workflow，作为Arsos Mac APP的精简版，[下载地址][4]

### 后续计划
和几个朋友聊了下，收集了一些需求，加上自己也有些想法，于是功能列表变得越来越长
![][image-6]
我目前对这个工具的定位是一个纯粹的搜索工具，作为所有库和组件的搜索入口，所以不仅仅限于前端的内容，后续还会添加OC、Swift、Python、Ruby、Go等内容。
最后，有朋友对这个工具有什么建议可以留言。







[1]:	https://github.com/liuyunclouder/alfredworkflow
[2]:	https://github.com/liuyunclouder/popclip_extension
[3]:	http://liuyunclouder.github.io/Arsos_site/
[4]:	https://github.com/liuyunclouder/alfredworkflow

[image-2]:	http://d.pr/i/UPad+
[image-3]:	http://d.pr/i/bk1C+
[image-4]:	http://d.pr/i/1fXDD+
[image-5]:	http://d.pr/i/GP2I+
[image-6]:	http://d.pr/i/15SO9+