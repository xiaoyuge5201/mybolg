---
title: CentOS安装Maven 3.8.6
comments: false
theme: xray
tags: maven
categories: linux
abbrlink: 51373
date: 2022-06-04 09:10:10
---
### 1. 下载
1. maven官网地址：https://maven.apache.org/download.cgi ， 右键复制链接地址，wget下载，或者下载到本地再上传到Centos
2. 可以从镜像仓库下载：https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.8.6/binaries/ ，
```shell
cd /usr/local/tools
https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.8.6/binaries/apache-maven-3.8.6-bin.tar.gz
```

### 2. 解压
```shell
tar -zxvf apache-maven-3.8.6-bin.tar.gz
```

### 3. 配置环境变量
```shell
vim /etc/profile
```
在末尾增加两行
```shell
export MAVEN_HOME=/usr/local/tools/apache-maven-3.8.6
export PATH=$PATH:$MAVEN_HOME/bin
```
编译生效
```shell
source /etc/profile
```
### 4. 验证是否配置成功
```shell
mvn -v
```
如果出现版本信息则配置成功

### 5. 配置本地仓库和镜像仓库
```shell
cd /usr/local/tools/apache-maven-3.8.6/conf
vim setting.xml
```
在相应注释位置添加如下两行：
```xml
<localRepository>/home/mavenRepository/</localRepository>
```
```xml
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```
