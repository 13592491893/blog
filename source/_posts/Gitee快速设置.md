---
abbrlink: 49456
title: Gitee快速设置
comments: true
toc: true
description: Gitee快速设置
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - git
tags:
  - git
  - gitee
  - config
date: 2021-4-21 16:00:00
---
# Gitee快速设置

## git全局设置

``` java
git config --global user.name "时鸿运"
git config --global user.email "459173919@qq.com"
```

## 创建 git 仓库

```java
mkdir tt
cd tt
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add origin https://gitee.com/gsshy/tt.git
git push -u origin master
```

## 已有仓库?

```java
cd existing_git_repo
git remote add origin https://gitee.com/gsshy/tt.git
git push -u origin master
```

