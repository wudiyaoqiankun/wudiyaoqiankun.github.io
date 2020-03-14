---
layout: post
title: "deepin 15.10+ 上安装 Python3"
subtitle: '记录安装Python少踩坑'
author: "Yao Qiankun"
header-img: "img/post-sample-image.jpg"
header-img-credit: "@WebdesignerDepot"
header-img-credit-href: "medium.com/@WebdesignerDepot/poll-should-css-become-more-like-a-programming-language-c74eb26a4270"
header-mask: 0.4
tags:
  - Python
---

Update:deepin 安装 Python3

---

## 1. 升级系统 & 安装依赖
&emsp;&emsp;打算编译安装以前，首先升级系统，安装必要的依赖。这样可以避免之后安装 Python 时缺少第三方库的问题。
```
sudo apt update
sudo apt install -y make build-essential libssl-dev zlib1g-dev
sudo apt install -y libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm
sudo apt install -y libncurses5-dev  libncursesw5-dev xz-utils tk-dev
```
<br>
## 2. 下载 Python 安装包并编译安装
### 2.1. 下载
&emsp;&emsp;这次计划安装 Python 3.6.8，相关页面：https://www.python.org/downloads/release/python-368/
```
wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tgz
```
<br>
### 2.2. 解压
```
tar xvf Python-3.6.8.tgz
```
<br>
### 2.3. 配置构建文件
```
cd Python-3.6.8
./configure --enable-optimizations
```
<br>
### 2.4. 编译安装
```
make -j8 && sudo make altinstall
```
<br>
## 3. 收尾工作
```
sudo make clean
```
<br>
至此 Python 已经安装完毕，可以在命令行输入python3.6查看。
