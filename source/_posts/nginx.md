---
title: nginx安装.md
date: 2021-07-23 11:40:44
tags: nginx
comments: false
---
# Nginx简介
- 2000年地洞，c语言编写
- 2004年开源
- 2011年成立商业公司
- 2013 发布商业版本Nginx plus
- 2019.5月F5 networks收购nginx
- 2019.12被Rambler集团起诉

##Nginx与其他web服务器对比
   1. Nginx与A pace HTTP server project区别
   2. Nginx 和tomcat区别
      - Nginx是HTTP Server，主要是用于访问一些静态资源，可以用做代理服务器
      - tomcat是Application Server应用服务器
   3. HTTP Server 和Application Server区别与联系

# Nginx安装

1. 安装nginx前首先要确认系统中是否安装了gcc 、pcre-devel、zlib-devel、openssl-devel

    ```shell
    #1、rpm包安装的，可以用 rpm -qa 看到，如果要查找某软件包是否安装，用 rpm -qa | grep "软件或者包的名字"
    #2、以deb包安装的，可以用 dpkg -l 看到。如果是查找指定软件包，用 dpkg -l | grep "软件或者包的名字"
    #3、yum方法安装的，可以用 yum list installed 查找，如果是查找指定包，用 yum list installed | grep "软件名或者包名"
    yum list installed | grep "gcc"
    ```
   ![image-20201210103251475](./nginx/image-20201210100736952.png)
2. 安装依赖包

    ```shell
    yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
    ```

3. 下载并解压安装包

    ```shell
    //创建nginx存放文件夹
    cd /usr/local
    mkdir nginx
    cd nginx
    #下载tar包
    wget http://nginx.org/download/nginx-1.13.7.tar.gz
    tar -xvf nginx-1.13.7.tar.gz
    ```

4. 配置

    ```shell
    cd nginx-1.13.7
    ./configure --prefix=/usr/local/nginx
    
    make
    make install
    ```

5. 测试是否安装成功

    ```shell
    ./sbin/nginx -t
    ```

    <img src="./nginx/image-20201210101238462.png" alt="image-20201210101238462" style="zoom: 67%;float:right;" />

6. 配置nginx.conf

    ```yml
    vim /usr/local/nginx/cong/nginx.conf
    
    #修改如下
    server {
      listen 80;
      server_name localhost;
    
      # 注意设定 root路径是有dist的
      location / {
        root /usr/local/webapp/dist;
        index /index.html;
      }
    
      #跨域 ip和port自行替换
      location /adminApi {
        proxy_pass http://ip:port;
      }
    
    }
    
    ```

7. 启动
   ```shell
       #启动nginx
       cd /usr/local/nginx/sbin
       ./nginx 
     ```

   **常用命令：**
   
   ```shell
       #修改配置后重新启动
       ./nginx -s reload
       #如果出现：nginx: [error] open() ＂/usr/local/nginx/logs/nginx.pid＂ failed
       /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
       #再次启动即可
       
       #查看nginx进程是否启动
       ps -ef|grep nginx
       
       #平滑启动nginx
       kill -HUP
       #主进程号或进程号文件路径 或者使用
       
       /usr/nginx/sbin/nginx -s reload
       
       #注意，修改了配置文件后最好先检查一下修改过的配置文件是否正 确，以免重启后Nginx出现错误影响服务器稳定运行。
       #判断Nginx配置是否正确命令如下：
       nginx -t -c /usr/nginx/conf/nginx.conf
       #或者使用
       /usr/nginx/sbin/nginx -t
       
       #重启
       nginx reload
       /usr/local/nginx/sbin/nginx -s reload 
       service nginx restart
       
       #启动
       ./nginx
       #关闭
       ./nginx -s stop
       
       
       #配置nginx开机自启动
       vim /etc/rc.d/rc.local
       
       #再文件中添加nginx启动地址
        
       touch /var/lock/subsys/local
       /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
       
       #设置开机自启动nginx
       /usr/local/nginx/sb/nginx
    ```
![image-20201210103251475](./nginx/image-20210606160947369.png)

启动后访问localhost 效果如下：
![image-20201210103251475](./nginx/image-20201210103251475.png)

# Nginx配置
```shell
...... 全局块
events {
	//events 块
}
http{
  ....  http全局块
	server+{
		location +[]
	}
}
```
### 配置内容规则
- 用#表示注释
- 每行配置的结尾需要加上分号
- 如果配置项值中包括语法符号，比如空格符，那么需要使用单引号或者双引号行括住配置项值，否则ngin x会报语法错误
- 单位简写：
   - K或者k千字节（kilo byte, KB）
   - M或者m兆字节（megabyte MB）
   - ms(毫秒)，s(秒)， m(分)， h(小时) ， d (天)， w（周）， M（月，包含30天），y（年）

