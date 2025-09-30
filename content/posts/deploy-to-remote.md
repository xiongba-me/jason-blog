---
title: '部署静态文件到远程服务器'
date: '2024-04-23'
tags: ['blog', 'ssh', 'remote','IDEA']
draft: false
summary: 使用简单的ssh命令部署静态博客
---
# 运行原理
- **博客是全静态文件部署的，在本地IDEA开发完以后，能快速部署到服务器上。**
- **使用了IDEA提供的external tools**
- **ssh免密登陆**

## SSH 免密登陆设置
生成 SSH 密钥对：尚未在本地系统上生成 SSH 密钥对，则需要生成一个。如果有就直接用现有的。
```shell
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
这将生成一个 4096 位的 RSA 密钥对，私钥保存在 ~/.ssh/id_rsa，公钥保存在 ~/.ssh/id_rsa.pub。
```shell
ssh-copy-id user@hostname
```
然后Copy到远程服务器上。

## IDEA 新建一个external tools 脚本命令
```shell
Arguments: $dir/deploy-blog.sh $HOST $ProjectFileDir$ $ProjectName$
Workspace: $ProjectFileDir$
```

## 客户端sh脚本,命名为deploy-blog.sh
```shell
#!/bin/sh
HOST=$1
PROJECT_PATH=$2
PROJECT_NAME=$3
TAR_FILE=$PROJECT_NAME.tar.gz
cd $PROJECT_PATH
echo '编译包'$TAR_FILE
yarn build
cd out
tar -cf $TAR_FILE *
echo '开始copy文件到远程目录'$TAR_FILE
scp -P $PORT  $TAR_FILE $USER_NAME@$HOST:/tmp/$TRA_FILE
ssh -p $PORT $USER_NAME@$HOST "sh -c './deploy.sh $PROJECT_NAME'"
```
## 服务端sh脚本,命名为deploy.sh
```shell
#!/bin/sh
PROJECT_NAME=$1
TAR_FILE=/tmp/$PROJECT_NAME.tar.gz
PROJECT_PATH=/opt/app/myweb/$PROJECT_NAME
echo 'deploy '$TAR_FILE
rm -rf $PROJECT_PATH/*
tar -xvf $TAR_FILE -C $PROJECT_PATH
```
## 使用方法
IDEA中两次shift键，找到刚才的 *external tools*，执行，就能自动复制文件，目标服务器自动解压。

