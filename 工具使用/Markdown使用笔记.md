
<center>Markdown使用笔记</center>

[TOC]

##一、安装编辑器
Markdown编辑器有很多个，网上一搜一大片。本文使用免费的sublime text。
###1、安装sublime text
到官网下载sublime text，安装。
###2、安装sublime text插件
####1）Package Control
这个是sublime text专门用来安装插件的。

找到菜单栏：```Preferences``` → ```Package Control``` → ```Package Control:Install Package```；

没有找到```Package Control```，可以在菜单```Tool```下找到```Install Package Control```。点击安装即可

####2）Markdown Editing
点击菜单```Preferences``` → ```Package Control```，进入控制台，输入```install package```,回车后进入搜索，输入```Markdown Editing```,选择安装即可。

Markdown Editing并不只是一个markdown的主题插件,它自定义许多markdown的快捷键

####3）Markdown Preview
安装参考步骤2）

这个插件不能实时预览,但你可以设置快捷键让它在浏览器中预览

在首选项->快捷键设置里添加以下设置：

```
{ "keys": ["alt+m"], "command": "markdown_preview", "args": {"target": "browser", "parser":"markdown"} },
```

在编辑markdown文件时可以通过按alt+m快速在浏览器中预览,虽然不如实时预览方便,但markdown的格式很简单,经常用的人并不需要频繁的预览,设计markdown的初衷本就是为了让编辑安心码字,不用过多考虑格式的问题

修改html输出文件路劲

在首选项->Package Setting->Markdown Preview->Settings的User部分添加以下设置

```
{
//文件生成在build目录下
"path_tempfile": "build"
}

```

即可在.md同目录生成build文件夹，生成的html就在其中。
 
####4）MarkdownLivePreview（选择安装）
安装参考步骤2）

MarkdownLivePreview可以实现实时预览,在首选项->Package Setting里修改MarkdownLivePreview的user配置文件,设置在打开时同步预览
将default.setting中的拷贝到user.setting中，修改下面对应的字段：

```
"markdown_live_preview_on_open": true

```

这样打开就有实时预览了，不过会有点卡顿的感觉，不推荐这样做。

##二、基本语法

参考教程，不同的编辑器，语法可能有些差别


1.块注释（blockquote）

>     通过在文字开头添加“>”表示块注释。（当>和文字之间添加五个blank时，块注释的文字会有变化。）

2.*斜体*

将需要设置为斜体的文字两端使用1个“*”或者“_”夹起来

3.**粗体**

将需要设置为斜体的文字两端使用2个“*”或者“_”夹起来

4.无序列表

在文字开头添加(\*, +, and -)实现无序列表。但是要注意在(\*, +, and -)和文字之间需要添加空格。（建议：一个文档中只是用一种无序列表的表示方式）

5.有序列表
使用数字后面跟上句号。或.（还要有空格）

6.链接（Links）

[Markdown教程](http://www.markdown.cn)

7.图片（Images）
```
内联方式：![alt text](https://goss.veer.com/creative/vcg/veer/800water/veer-169262069.jpg "Title")
```

![alt text](https://goss.veer.com/creative/vcg/veer/800water/veer-169262069.jpg "Title")

```
引用方式：![alt text][id] 

[id]: https://goss.veer.com/creative/vcg/veer/800water/veer-121795711.jpg "Title"
```
![alt text][id] 


[id]: https://goss.veer.com/creative/vcg/veer/800water/veer-121795711.jpg "Title"

8.字体、字号、颜色

<font face="黑体">我是黑体字</font>

<font face="微软雅黑">我是微软雅黑</font>

<font face="STCAIYUN">我是华文彩云</font>

<font color=#0099ff size=12 face="黑体">黑体</font>

<font color=#00ffff size=3>null</font>

<font color=gray size=5>gray</font>

##三、样式更改
###1、Markdown生成左边框目录
1)首先你要写一个格式正确的Markdown文本, 主要是每一级标题要使用正规的Markdown语法：

```
# 这是一级标题
## 这是二级标题
### 这是三级标题
###### 这是六级标题
```

2)使用[TOC]生成一个自带的目录，[TOC]是Markdown自动生成目录的方法，建议将目录生成在文章最上边，方便下面操作。

3)CSS修改，打开生成的HTML文件，修改CSS样式。修改结构如下：

```
<html>
    <head>
        <style>
            body  {
                /*width: 45em;*/
                border: 1px solid #ddd;
                outline: 1300px solid #fff;
                margin: 16px auto;
            }
            .left-div {
                width:17%;
                float:left;
                padding-right: 10px;
                position: fixed;
                overflow-y:scroll;
                height: 80%
            }
            .right-div {
                width:80%;
                float:right;
                padding-left: 10px;
            }
        </style>
    </head>
    <body>
        <!--article标签中就是我们编写的文本内容-->
        <article>
            <!--这里将生成的class改成left-div,便于理解-->
            <div class="left-div">这里是[TOC]命令生成的目录</div>
            <!--这里新增个<div class="right-div"></div>,将正文部分包裹-->
            <div class="right-div">这里是正文部分</div>
        </article>
    </body>
</html>
```

要是能修改生成的模板就好了，就不用每次生成都去修改这个样式了。

##参考资料
[Markdown教程](http://www.markdown.cn)

[Markdown语法说明](http://wowubuntu.com/markdown/)



