---
title: VScode WSL开发环境配置
date: 2019-08-10 14:40:08
tags: 
    - Linux
categories: 
    - 操作系统
---

Windows想用到Linux系统只能WSL了。

<!-- more -->

读研期间日常写文档用windows word，工作时使用linux码代码需要来回切换操作系统，幸亏遇到wsl ubuntu解决一大难题，但配置好开发环境（[使用VScode和Cmake搭建嵌入式开发环境](http://jianliang-shen.cn/2019/07/22/%E4%BD%BF%E7%94%A8VScode%E5%92%8CCmake%E6%90%AD%E5%BB%BA%E5%B5%8C%E5%85%A5%E5%BC%8F%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/)），使用windows版本的VScode有一大弊端--无法查看linux的头文件内容，无法跳转相关的函数和变量定义，简直扎心！无意中发现插件中有一个remote wsl的，真的让人awsl！微软真香！具体可以参考[搭配 VS Code Remote 远程开发扩展在 WSL 下开发](https://www.cnblogs.com/nczitzk/p/develop-in-wsl-with-vscode-remote.html)。
<!-- more -->  
## 注意事项
1. 在运行remote wsl时，需要重新安装插件，这些插件和Windows VScode不同；
2. 在命令输入`remote wsl:new window`进入wsl vscode；
3. VScode WSL是在linux下工作的，新建的终端也为linux bash，代码补全、头文件均为linux版本；
4. 关于git中文显示乱码问题，参考：  
   [ubuntu下设定系统locale，支持中文zh_CN.UTF-8](https://blog.csdn.net/deepxl/article/details/17802451)
   [ubuntu命令行下中文乱码的解决方案](https://www.cnblogs.com/york-hust/archive/2012/03/27/2419582.html)
   语言支持安装：`sudo apt install $(check-language-support) --fix-missing`  
   git关闭编码设置：`git config --global core.quotepath false`


