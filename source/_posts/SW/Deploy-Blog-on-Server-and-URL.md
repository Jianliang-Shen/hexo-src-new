---
title: hexo部署博客并上传到域名上
date: 2019-06-29 09:07:19
index_img: /img/post_pics/index_img/hexo_index.png
tags: 
    - Hexo
    - Linux
categories: 
    - Software
---

文章转载来自[使用 Hexo + Github 搭建自己的博客（图文教程）](https://blog.csdn.net/qq_40147863/article/details/84942914)  
替换主题参考[Hexo 安装和替换主题、自定义博客主题](https://blog.csdn.net/qq_40147863/article/details/84946894)
由于github部署的博客连接较慢，因此在购买腾讯云服务器之后将博客内容部署到腾讯云上，特此记录一下过程。

<!-- more -->

- [搭建过程](#搭建过程)
  - [安装git node.js](#安装git-nodejs)
  - [新建仓库](#新建仓库)
  - [安装Hexo](#安装hexo)
  - [修改`_config.yml`](#修改_configyml)
  - [建立ssh密钥](#建立ssh密钥)
  - [维护过程](#维护过程)
- [github自动推送](#github自动推送)
- [云服务器部署](#云服务器部署)
  - [创建git用户和仓库](#创建git用户和仓库)
  - [使用nginx反向代理](#使用nginx反向代理)
  - [开放云服务器端口](#开放云服务器端口)
  - [设置Hexo仓库](#设置hexo仓库)
  - [配置域名解析](#配置域名解析)
  - [域名备案](#域名备案)

## 搭建过程

### 安装git node.js

查看版本命令用  

```bash
npm -v
node -v
```

### 新建仓库

新建一个repository，名称为 `name.github.io`

### 安装Hexo

首先安装cnpm

```bash
# 全局安装，这样hexo才能运行
cnpm install hexo -g
cnpm list -g
cnpm uninstall -g hexo

# 会卡住
hexo init
# 手动安装依赖
cnpm install -f
hexo s

# 安装更多依赖
cnpm install hexo-util
cnpm install moment
cnpm install hexo-pagination
cnpm install @adobe/css-tools
cnpm install hexo-deployer-git
cnpm install hexo-helper-live2d
cnpm install live2d-widget-model-epsilon2_1

# or
npm install hexo -g
npm list -g
npm uninstall -g hexo
```

``` bash
npm install hexo -g
hexo -v
hexo init
npm install
hexo g
hexo s  #查看本地新建的hexo
npm install hexo-deployer-git --save
```

### 修改`_config.yml`

``` yml
deploy:
  type: git
  repository: git@github.com:Jianliang-Shen/Jianliang-Shen.github.io.git
  branch: master

```

### 建立ssh密钥

``` bash
ssh-keygen -t rsa -C "mail@xx.com"
```

并将默认保存位置的`id_rsa.pub`内容存放到github网站中的SSH密钥中。

### 维护过程

``` bash
hexo new post "blog-name"
hexo d -g
```

## github自动推送

1. 创建`.github\workflows\deploy.yml`
2. 在仓库添加私钥，SSH_PRIVATE，找一个本地一样的私钥
3. 注意，packages.json也要提交

yaml文件内容：

```yaml
name: Deploy                      # Actions 显示的名字，随意设置

on: [push]                        # 监听到 push 事件后触发

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - name: Checkout              # 拉取当前执行 Actions 仓库的指定分支
      uses: actions/checkout@v2
      with:
        ref: main

    - name: Update Submodule      # 如果仓库有 submodule，在这里更新，没有则删掉此步骤
      run: |
        git submodule init
        git submodule update --remote

    - name: Setup Node            # 安装 Node 环境
      uses: actions/setup-node@v1
      with:
        node-version: "14.x"

    - name: Hexo Generate         # 安装 Hexo 依赖并且生成静态文件
      run: |
        rm -f .yarnclean
        yarn --frozen-lockfile --ignore-engines --ignore-optional --non-interactive --silent --ignore-scripts --production=false
        rm -rf ./public
        yarn run hexo clean
        yarn run hexo generate

    - name: Hexo Deploy           # 部署步骤，这里以 hexo deploy 为例
      env:
        SSH_PRIVATE: ${{ secrets.SSH_PRIVATE }}
        GIT_NAME: Jianliang Shen
        GIT_EMAIL: 2823626197@qq.com
      run: |
        mkdir -p ~/.ssh/
        echo "$SSH_PRIVATE" | tr -d '\r' > ~/.ssh/id_rsa
        chmod 600 ~/.ssh/id_rsa
        ssh-keyscan github.com >> ~/.ssh/known_hosts
        git config --global user.name "$GIT_NAME"
        git config --global user.email "$GIT_EMAIL"
        yarn run hexo deploy
```

## 云服务器部署

服务器安装CentOS 7操作系统，首先安装Nodejs和git：  
  
```bash
yum install nodejs
yum install git
```
  
### 创建git用户和仓库  
  
```shell
adduser git
chmod 740 /etc/sudoers
vim /etc/sudoers
=== root    ALL=(ALL)     ALL
+++ git     ALL=(ALL)     ALL
chmod 400 /etc/sudoers

sudo passwd git
su git
mkdir ~/.ssh
vim ~/.ssh/authorized_keys
#然后将电脑中执行 C:\Users\28236\.ssh\id_rsa.pub ,将公钥复制粘贴到authorized_keys
linux: cat ~/.ssh/id_rsa.pub
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
ssh -v git@SERVER

# su git  切换用户进行后续操作
cd /home/git/
mkdir -p projects/blog # 把项目目录建立起来
mkdir repos && cd repos
git init --bare blog.git # 创建仓库
cd blog.git/hooks
vim post-receive # 创建一个钩子
+++ #!/bin/sh
+++ git --work-tree=/home/git/projects/blog --git-dir=/home/git/repos/blog.git checkout -f
chmod +x post-receive # 添加可执行权限
exit # 返回到root用户
chown -R git:git /home/git/repos/blog.git # 给git用户添加权限
```

### 使用nginx反向代理  
  
安装nginx及使用：

```bash
yum install nginx
nginx -s reload  #重载配置文件
nginx -s stop
sudo service nginx restart
systemctl enable nginx.service  #设置开机自启动
```

配置nginx，修改/etc/nginx/nginx.conf：

```json
user root;         //用户
...
server {
        listen       80;
        # listen       [::]:80 default_server;
        server_name  www.jianliang-shen.cn jianliang-shen.cn;   //个人域名
        root         /home/git/projects/blog;                  //仓库位置

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```

保存后重新部署。在公网中尝试连接，一般来说是失败。  

### 开放云服务器端口

因为腾讯云服务器并没有开放端口无法外部ping通，因此需要在“安全规则中”开放端口，如下所示：  
  
![fig1](/img/post_pics/Front/front1.png)

### 设置Hexo仓库

```json
deploy:
    type: git
    repo: git@server_ip:/home/git/repos/blog.git    
    branch: master
```

提交更新：

```bash
hexo d -g
```

### 配置域名解析

例如aliyun的域名解析设置如下：  
  
![fig2](/img/post_pics/Front/front2.png)

### 域名备案

各类云服务器在绑定域名时都需要备案，否则无法使用，例如我用的阿里云域名和腾讯云服务器，就需要在腾讯云中登记公网ip绑定的域名。
