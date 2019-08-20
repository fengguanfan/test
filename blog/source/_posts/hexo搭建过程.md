---
title: hexo搭建过程
date: 2019-07-20 20:42:21
categories: 
    - hexo
tags: 
    - hexo
mp3: /musics/山外小楼夜听雨.mp3
cover: /images/main/大海.jpg
---

# 第一步 hexo安装
1.安装nodejs  地址：nodejs.org
2.安装cnpm淘宝镜像源 命令： npm install -g cnpm –registry=https://registry.npm.taobo.org 
3.安装hexo 命令：cnpm install -g hexo-cli 检查版本 hexo -v 检查是否成功
4.创建blog文件夹 命令：mkdir blog 再进入目录 cd blog (如果是windows系统,直接创建文件夹,注意命令要管理员权限)
5.初始化博客 命令：sudo hexo init 注意要进入blog文件夹
6.启动博客 命令：hexo s 在blog文件夹里 成功后默认地址：localhost:4000
7.新建一个博客 命令：hexo n “我的第一个文章”  进入文章所在的目录 命令: cd so urce/_posts/ 修改md文件内容
8.退回到blog文件夹中 清理hexo命令:hexo clean 生成hexo命令:hexo g 启动hexo命令：hexo s 

# 第二步 github 托管
1.安装git mac系统有无需安装(进入root命令: sudo su)，windows系统需要安装  地址：https://git-scm.com/downloads
2.注册登入github 地址:github.com
3.新建一个新的存储库 （+号下拉框的第一个） 注意:存储库的命名一定要是的呢称+github.io 如：fengguanfan.github.io
4.在blog文件夹下安装git插件 命令：cnpm install hexo-deployer-git --save
5.配置_config.yml文件 打开文件在最后添加如下配置：
		type: git
		repo: https://github.com/fengguanfan/fengguanfan.github.io
		branch: master
6.部署到远端 命令: hexo d 中间输入git的账号和密码
(windows需要在git配置里加上账号邮箱和用户 
命令：git config --global user.name "xxx" git config --global user.email "xxx@xxx.com")
7.成功后用存储库的名字直接访问如：fengguanfan.github.io

# 第三步 主题更换
1.在主题官网上找到自己喜欢的主题 地址: https://hexo.io/themes/
2.进入他的git主题网址,根据提示，更换主题
