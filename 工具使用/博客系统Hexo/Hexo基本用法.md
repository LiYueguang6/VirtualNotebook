# Hexo基本用法

## 一、简介

Hexo是一个基于Node.js的轻量级博客系统

## 二、搭建



## 三、开始

### 1.新建博客
```powershell
hexo new [layout] <title>
```
也可以直接在\_drafts新建markdown文件，就是草稿了，在\_posts新建markdown文件，就是发布的文章了。在顶部加上相关的描述。
```
---
title: hexo——轻量、简易、高逼格的博客
date: 2018-08-31 17:54:54
tags:
    - IT技术
    - 前端
---
```

### 2.发布
```hexo generate```或者```hexo g```生成页面
```hexo deploy```或者```hexo d```上传

### 3.预览

```hexo s```执行后访问localhost:4000预览

## 四、进阶用法

### 1.主题

#### 1.1主题安装

#### 1.2推荐主题

* yilia：https://github.com/litten/hexo-theme-yilia.git

## 五、其他问题

### 1.每次提交需要输入用户名和密码

在生成SSH秘钥之后，还会出现这个问题，那么应该是_config.yml的repo选项使用的还是Https的协议，更换成SSH协议的链接即可，链接在github上可以找到。