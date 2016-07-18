---
title: 我是如何在Deepin Linux系统上工作的
date: 2016-07-15 15:29:25
tags: 操作系统
---

## 背景
　　我是一名java工程师，一直以来都以Eclipse进行java开发。从2015年底开始在Linux上工作，期间安装过Ubuntu\Deepin\CentOS等系统，最终确定在Deepin下工作是被其界面吸引。
　　最开始用的Deepin还是基于Ubuntu开发的，从15版本开始是基于Debian上定制开发。尝试请下载：[下载地址](https://www.deepin.org/download.html) 

下面说说我常用的软件及配置：

### 基础工具
*  以下是我常用的一些工具
``` bash
sudo apt-get install git xclip htop guake tree tmux
```
> *git* 用来管理源代码的版本及分支
   *xclip* 用来操作剪贴板
   *htop* top加强管理工具
   *guake* 可以后台运行的终端，F12可在后台放音乐。
   *tree* 用来显示目录
  *tmux* 用来管理终端连接

### 系统自带软件
* **CrossOver** 用于运行widows相关的软件，如QQ
* **WPS** 用于文档、演示文稿、电子表格操作
* **深度商店** 用于快速安装仓库自带软件
* **深度音乐/深度影院/深度截图** 用于娱乐、截图
* **Gedit** 用于文本编辑
* **深度终端** 用于命令操作，可调整为zsh，并用oh-my-zsh加强
* **文件管理器** 用于文件浏览、查看、删除
* **搜狐拼音/五笔拼音**　用于输入法
* **Chrome浏览器** 用于上网及网页开发调试
  -  常用插件有：
[*Axure RP Extension for Chrome*]() 用于查看产品原型
[*JSONView*]() 用于查看格式化Json
[*Momentum*]() 用于默认开tab页使用优美图片
[*Octotree*](http://www.oschina.net/p/octotree) 用于树状查看github代码
[*SwitchyOmega*](http://www.oschina.net/p/switchyomega) 用于代理设置
[*Postman*](http://www.oschina.net/p/postman) 用于API接口测试        
[*今天日期*]() 用于查看当前的时间

### 常用工具软件
* 有道词典 翻译软件，用于专业词汇查找。
* 为知笔记 记录软件，用于工作总结。
* VirtualBox 虚拟机软件，用于Windos虚拟、Android虚拟。
* Wireshark 抓包软件，用于协议分析、调试

### 附加软件
* 微信网页版
* 钉钉
* Firefox

### JAVA开发环境
#### IntelliJ IDEA
    长期使用Eclipse后再用IDEA就很少用回Eclipse了，毕竟IDEA功能比Eclipse强大很多。
  - 集成了Maven环境，可以修改配置路径指向自己下载的maven。
  - 集成了Git环境，方便进行分支切换、代码提交。
  - 调试功能强大，用Visual Studio的同学都会佩服其调试功能，IDEA也是如此。
  - 更高的开发效率，默认深色主题，重构、跳转定义、删除/复制行方便。
  - 集成命令行终端，相当多的快捷键，并可配合终端操作。

#### Dbeaver
    用于数据库操作
### Sublime Text 3
    用于代码查看及编译
### Visual Studio Code
    用于golang开发
