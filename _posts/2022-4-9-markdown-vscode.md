---
layout: article
tags: 工具
title: 在VSCode中玩转MarkDown
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

VSCode中几个好用的MarkDown插件

<!--more-->

## 预览：Markdown Preview Enhanced

可以实时的编写并预览效果，安装插件后在md文件右键就可以打开预览窗口，在预览窗口中右键可以选择导出PDF等格式的文件，还可以修改预览样式

美中不足是预览样式不能够应用到到保存的文件中

## 保存：Markdown PDF

安装插件后在md文件右键就可以保存成pdf，html，png和jpeg四种格式

支持通过外部文件修改导出的样式：

[![LiMmK1.png](https://s1.ax1x.com/2022/04/09/LiMmK1.png)](https://imgtu.com/i/LiMmK1)

将下载好的css文件`绝对路径`添加到设置中即可

样式可以在[这里](https://theme.typora.io/)找

## 生成目录：Markdown TOC

安装插件后打开要生成目录的md文件，通过快捷键`Ctrl+Shift+P`打开搜索框，输入`Generate TOC for markdown`就可以自动生成目录，并包含标题跳转功能

使用时发现最高的目录级别是两个#号，如果有一个#号的标题会不识别

