---
layout:	
title: 使用Hexo+GitHub Pages搭建个人博客	
date: 2018-05-05 20:50
updated: 2018-05-05 21:08
comments: true
tags:
- GitHub Pages
- Hexo
categories:
- 版本控制	
permalink: 
---

[TOC]

# 1. 基础设置

## 1.1 创建GitHub仓库

创建一个名为widgrs.github.io的仓库，用于存放发布的文章信息；创建一个名为MyHexo的仓库，用于存放个人的Hexo数据。

<!-- more -->

## 1.2 安装Git

```shell
# 安装Git
sudo apt-get install git
# 配置Git
git config --global user.name "Widgrs"
git config --global user.email "jichliu@163.com"
# 查看Git全局设置
git config --global -l

# 查看是否有SSH Keys
ls -al ~/.ssh
# 如果没有id_rsa文件，则生成一对密钥对
ssh-keygen -t rsa -b 4096 -C "Widgrs's Key"
# 更改id_rsa文件权限
chmod 644 id_rsa
# 将id_rsa.pub文件中的内容复制到GitHub SSH Keys设置中

# 测试是否能成功连接到GitHub
ssh -T git@github.com
# 如果成功，返回结果为：
# Hi Widgrs! You've successfully authenticated, but GitHub does not provide shell access.
# 如果提示Failed to add the host to the list of known hosts (/home/ubuntu/.ssh/known_hosts)
# 则在.ssh目录下新建一个known_hosts文件并将其权限更改为766
```

## 1.3 安装Node.js和Hexo

```shell
# 安装Node.js的最佳方式是使用nvm
wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
# 安装完成后重启终端并执行下列命令即可安装Node.js
nvm install stable
# 查看Node.js是否安装成功
node -v

# 安装Hexo
npm install -g hexo-cli
```

## 1.4 测试Hexo是否可用

```shell
# 初始化测试文件夹~/Hexo
hexo init ~/Hexo
cd ~/Hexo
# 安装依赖包
npm install
# 启动本地服务器，可以通过http://localhost:4000/访问
hexo server
```

# 2. 创建站点

## 2.1 创建站点

在一个新目录中初始化MyHexo，然后将该目录和github.com:Widgrs/MyHexo项目关联。

```shell
cd ~/Code/GitHub
hexo init MyHexo
cd MyHexo
npm install
# 为了能够使Hexo部署到GitHub上，需要安装一个插件
npm install hexo-deployer-git --save
# 可以使用hexo server命令启动本地服务验证是否正常

# 将MyHexo目录和github.com:Widgrs/MyHexo项目关联
# 当前目录为~/Code/GitHub/MyHexo
git init
git remote add origin git@github.com:Widgrs/MyHexo.git
# 本地master分支关联远程的master分支
git branch master origin/master
git checkout master
# 将空仓库github.com:Widgrs/MyHexo中的内容拉到本地
git pull origin
```

## 2.2 修改_config.yml配置文件

```yaml
# 文件路径：~/Code/GitHub/MyHexo/_config.yml，该文件是网站的全局配置文件
# Site
title: Widgrs's Blog
subtitle: 每一个不曾起舞的日子都是对生命的辜负！
description: 默默滴躲在墙角写Bug的代码狗！
keywords:
author: Widgrs
language: zh-CN
timezone:

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://widgrs.github.io
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
# 这一行暂时不改，下载NexT主题之后再改
theme: next 

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:Widgrs/widgrs.github.io.git
  branch: master
```

## 2.3 测试&部署

在部署之前可以使用hexo server命令启动本地服务器预览。

第一次部署之前需要提交一次代码，否则会出错，因为hexo init新添加的文件和文件夹并没有提交到GitHub。

```shell
# 提交代码
git add -A
git commit -m "Initial Widgrs's Blog With Hexo"
git push -u origin master

# 部署页面
# 由于Git不能跟踪.deploy_git文件夹，所以将.deploy_git添加进.gitignore将其排出
hexo generate
hexo deploy
```

# 3. 使用NexT主题

## 3.1 安装NexT主题

```shell
cd ~/Code/GitHub/MyHexo
# 当前分支为master
git clone https://github.com/theme-next/hexo-theme-next themes/next
# 备份原始的配置文件
cp _config.yml _config.yml.old
# 为了解决Git不能跟踪某些文件的问题，修改部分文件或文件夹名称
mv themes/next/.git themes/next/.widgrs.git
```

启用主题：将~/Code/GitHub/MyHexo/_config.yml 文件中theme字段的值改为next。

验证主题：在验证之前最好使用hexo clean命令清除Hexo的缓存。

主题设定：将~/Code/GitHub/MyHexo/themes/next/_config.yml文件中scheme字段改为Pisces。

说明：由于themes/next目录下有.git文件夹，导致Git不能跟踪next目录，会将整个next目录当成一个文件，解决办法是将next目录下的.git文件夹重命名即可，比如重命名为.widgrs.git。**注意，一定要在添加next主题之后、使用git add -A命令之前将next/.git文件夹重命名，否则Git依然把next目录当成一个文件。**

（此处提交一次代码）

## 3.2 设置NexT

参照官方文档设置即可：[NexT使用文档](http://theme-next.iissnan.com/)

主要内容有：

* 根据需要修改`_config.yml`、`themes/next/_config.yml`配置文件。
* 在source目录下新建categories、archives、tags、about目录并上传对应的index.md文件。
* 在source目录下新建uploads目录并上传头像、打赏二维码等图片。

# 4. 绑定域名

由于页面部署实际上是将~/Code/GitHub/MyHexo/public目录下的所有文件推送到widgrs.github.io仓库的master分支，所以在该目录下新建一个文件CNAME，文件内容为要绑定的域名widge.cn，然后将该域名解析指向widgrs.github.io即可。

# 5. 疑难杂症



# 6. 布局优化

## 6.1 引用链接颜色设置

在~/Code/GitHub/MyHexo/themes/next/source/css/_common/components/post/post-expand.styl文件.posts-expand .post-body字段中添加一句a { color: #258fb8; }即可改变文章中引用链接的颜色。

# 7. 参考资料

* [GitHub Pages + Hexo搭建博客](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/#more)
* [Hexo文档](https://hexo.io/zh-cn/docs/index.html)
* [hexo-theme-next 源代码](https://github.com/theme-next/hexo-theme-next)
* [NexT使用文档](http://theme-next.iissnan.com/)
* [Next主题(Hexo)](https://www.jianshu.com/p/5d5931289c09)
* [畅言](http://changyan.kuaizhan.com/)
* [百度统计](http://tongji.baidu.com/web/welcome/login)











