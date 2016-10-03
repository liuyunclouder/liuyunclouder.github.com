
---
title: Popclip的JSON格式化扩展
date: 2016-09-29 20:00:00
---

作为一个MAC党，不好好利用MAC的神兵利器，简直就是罪过。Alfred、Dash、Ulysses、SnippetsLib、Mindnode等大名鼎鼎的效率神器自然不用提了，Popclip更是一个每天都会使用上百遍的好帮手。


## Popclip？

简单来说，Popclip就是一个对选中的内容作快速处理的工具，比如直接搜索选中的内容、从选中的内容生成二维码、计算选中的内容的字数等，除此之外，还能自定义扩展来实现你想要的功能。

这是我的Popclip扩展：
![][image-1]

如果你还没装Popclip，马上停下来，去安装一个，再继续看下去；

如果你不知道Popclip是什么，马上停下来，去看下这篇[测评][1]，再继续看下去。

## JSONizer的来由

平时经常需要对一坨字符进行格式化，那时每次都需要复制、打开[jsbeautifier.org][2]、粘贴、点击格式化按钮，碰到网络不好的情况还要等半天，如果没网络，更是头疼。

后来改用sublime的插件CodeFormatter，也能比较方便地快速格式化，但还是有个点让我不开心：CodeFormatter要求必须先把需要格式化的内容保存在一个后缀为.json的文件中，才能识别并格式化。

由于用Popclip已经好一段时间了，很享受它提供的便利，于是就想装个JSON格式化扩展，搜了一下，发现竟然没有，于是就萌生了自己写一个的想法。


## 动手
JSON格式化的lib都已经很成熟了，正好在[jsbeautifier.org][3]上看到有提供python的一个lib。

Popclip的扩展没有Alfred的workflow能提供的功能多而复杂，相应地也容易上手。参照[TUTS][4]上的这篇教程，几分钟就搞定了大致框架。

接下来就简单了，把依赖的几个lib依赖配好，基本文件布局如下：
![][image-2]

注：editorconfig、six.py是jsbeautifier的依赖项。

最后，测试效果完美：
![][image-3]

[ 下载入口 ][5]，希望能帮到需要的朋友。

## 总结

目前，需要先将需要格式化的内容拷贝到编辑器中，然后再选中才能格式化。其实还能改进一下，不需要拷贝，直接在内容来源上，比如浏览器中，选中需要格式化的字符并格式化，直接把格式化后的内容写入系统剪贴板。后续有时间可以研究下。





[1]:	http://sspai.com/25483
[2]:	http://jsbeautifier.org/
[3]:	http://jsbeautifier.org/
[4]:	http://computers.tutsplus.com/tutorials/popclip-scripting-extensions--mac-55842
[5]:	https://github.com/liuyunclouder/popclip_extension

[image-1]:	http://d.pr/i/sfTc+
[image-2]:	http://d.pr/i/V4JL+
[image-3]:	http://d.pr/i/hLal+