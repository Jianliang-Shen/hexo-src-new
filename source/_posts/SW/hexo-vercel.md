---
layout: post
title: 使用 Vercel 部署博客和设置 SSL 证书
date: 2024-07-25 18:08:04
index_img: /img/post_pics/vercel/index.png
tags: 
    - Hexo
    - Linux
categories: 
    - Software
---

详细说明Vercel的一些高级使用方法！

<!-- more -->

参考

- [Vercel 部署高级用法教程](https://hexo.fluid-dev.com/posts/hexo-vercel/)
- [vercel](https://vercel.com/)

首先创建一个新的github.io仓库，推送博客。部署好后，即可点击github.io链接访问page。在原先的方案中，我使用GitHub DNS进行域名解析，在仓库的page添加domain即可。

![Github DNS](/img/post_pics/vercel/github_dns.jpg)

在无证书情况下，可以访问域名（不能用https，即www），但是经常报DNS解析有错，这样就没法使能https，解决不了证书问题。

![DNS error](/img/post_pics/vercel/github_dns_error.jpg)

解决：改用新的vercel + ssl模式，停用掉github DNS。

## 申请免费证书

![CA](/img/post_pics/vercel/ca.jpg)

申请后绑定域名，会自动增加解析：

![CA 证书域名解析](/img/post_pics/vercel/ssl_dns.jpg)


## Vercel 导入项目

![导入github.io项目](/img/post_pics/vercel/prj.png)

## 设置域名

这里填域名的时候会要求去设置域名解析，解析后就可以验证DNS

![vercel域名解析](/img/post_pics/vercel/ali.png)

![域名](/img/post_pics/vercel/domain.png)


## 设置区域

发现解析后需要梯子才能访问，把区域设置为香港：

![设置区域](/img/post_pics/vercel/function.png)
