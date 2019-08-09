---
layout: post
title: "Harbor 部署笔记"
subtitle: '使用 docker 部署 Harbor'
author: "Yao"
header-style: text
tags:
  - Vim
---

Update:最终我终于摸到了 Harbor 的皮毛

---
1. Harbor依赖

  Harbor被部署为几个Docker容器，因此Harbor可以部署在任何支持Docker的发行版上。安装Harbor目标主机需要安装python、docker和docker-compose。

硬件要求

资源	大小	描述
CPU	最少2核	推荐4核
内存	最少4GB	推荐8GB
硬盘	最少40GB	推荐160GB
软件要求

软件	版本	描述
Python	2.7或更高版本	Python安装指南
Docker engine	1.10或更高版本	Docker安装指南
Docker Compose	1.6.0或更高版本	Docker Compose 安装指南
Openssl	推荐最新版本	为Harbor生成证书和密钥
2. 安装

  Harbor官方支持在线安装和离线安装两种方式，因为官方网络不稳定，在选安装可能会失败，一般选择离线安装。

下载安装文件 
wget https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-offline-installer-v1.7.5.tgz
解压 
tar xvf harbor-offline-installer-v1.7.5.tgz
配置 
cd到解压后的目录中，存在harbor.cfg文件，此文件为Harbor的配置文件，将hostname修改为主机IP，其余配置选项下一节细述。
安装 运行install.sh文件，将会进行自动安装。
  安装完成以后，在浏览器中输入上一步中配置的hostname，默认端口为80，即可进入Harbor登陆界面，默认管理员账号admin，密码Harbor12345 。

3. 配置文件介绍

  下文列出一些常用的配置，可以根据项目需要更改配置，更多配置信息请访问完整配置。

harbor.cfg

# 版本号
_version=1.7.0

# 主机IP或域名，不可使用localhost或127.0.0.1
hostname=39.97.185.26

# ui界面访问协议,可设置为http和https，默认http
ui_url_protocol = http

# 最大复制工作数
max_job_workers = 10

# 当此属性打开时，准备脚本将为注册表令牌的生成/验证创建私钥和根证书。当密钥和根证书由外部源提供时，将此属性设置为off
customize_crt = on

# SSL证书和密钥路径，仅当协议设置为https时使用
ssl_cert = /data/cert/server.crt
ssl_cert_key = /data/cert/server.key

# 用于在复制策略中加密或解密远程注册表的密码的密钥路径
secretkey_path = /data

# 日志文件数量
log_rotate_count = 50

# 日志文件总大小，单位可以为K、M、G、
log_rotate_size = 200M

# 邮箱设置，Harbor需要这些参数向用户发送重置密码的邮件，
email_server = smtp.mydomain.com
email_server_port = 25
email_identity =
email_username = sample_admin@mydomain.com
email_password = abc
mail_from = admin sample_admin@mydomain.com
email_ssl = false

# 默认管理员密码，仅在第一次启动时生效
harbor_admin_password = Harbor12345

# 身份认证类型，默认时db_auth，即凭据存储在数据库，若使用LDAP身份验证请设置为ldap_auth
auth_mode：db_auth

# 是否开启用户自注册账户，当禁用时，只有管理员可以创建新用户
self_registration = on

# 令牌服务创建的令牌到期时间，单位为分钟
token_expiration：30

# 控制用户是否有权限创建项目，设为everyone允许用户创建项目，设置为adminonly只有管理员可以创建用户
project_creation_restriction = everyone

  Harbor默认使用80端口，可以通过修改docker-compose.yml文件修改端口。还需修改harbor.cfg中的hostname配置，在ip后加上端口39.97.185.26:8888。

docker-compose.yml

    image: goharbor/nginx-photon:v1.6.0
    container_name: nginx
    restart: always
    volumes:
      - ./common/config/nginx:/etc/nginx:z
    ports:
      - 8888:80  # 将Harbor的80端口映射到主机的8888端口
      - 443:443
    depends_on:
      - postgresql
      - registry
      - core
      - portal
      - log
    logging:
      driver: "syslog"
      options:  
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "proxy"
4. 使用介绍

4.1 UI界面访问

  使用浏览器访问http://39.97.185.25:8888即可进入Harbor登陆界面，默认管理员账号admin，密码Harbor12345，输入账号密码进入UI操作界面。


  管理员可以通过UI界面进行创建项目、创建用户、分配用户权限、查看日志等操作，详情请查看Harbor用户指南。

4.2 通过docker上传\下载镜像

  docker login命令默认登录DockerHub服务器，使用docker login 39.97.185.26:8888即可登录Harbor服务器。

  登录之前还需将Harbor服务器加入到Docker的信任列表中，修改/etc/docker/daemon.json文件，如此文件不存在则创建，加入键值对"insecure-registries":["39.97.185.26:8888"]。

  在终端输入docker login 39.97.185.26:8888命令，根据提示输入用户名和密码，显示login succeed即说明登录成功。

  通过UI界面创建项目后，即可往项目中上传Docker镜像，假设创建的项目为test,上传之前需要先给镜像打标记，在终端执行命令：

docker tag SOURCE_IMAGE[:TAG] 39.97.185.26/test/IMAGE[:TAG]

  打完标记以后，执行以下命令即可完成上传：

docker push 39.97.185.26/test/IMAGE[:TAG]

  从Harbor拉取镜像执行命令：

docker pull 39.97.185.26/test/IMAGE[:TAG]