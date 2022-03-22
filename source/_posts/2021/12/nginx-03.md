---
title: Nginx基础篇（三）实现虚拟主机
comments: false
tags: nginx
categories: nginx
abbrlink: 59090
date: 2021-12-05 16:31:25
translate_title: nginx-install
---
### 1. 虚拟主机Virtual Host
一种在单一主机或主机群上，实现多网域服务的方法，可以运行多个网站或服务的技术，虚拟主机之间完全独立，并可由用户自行管理虚拟并非指不存在，而是指空间是由实体的服务器延伸而来，其硬件系统可以是基于服务器群，或者单个服务器

使用域名访问虚拟主机，虚拟主机会给一个文件路径，然后部署自己的内容；访问域名时就会访问改文件夹下的某 个资源

### 2. 使用Nginx配置虚拟主机
1. 在nginx下建立一个ygb的文件夹，里面新建一个index.html
2. 在nginx.conf配置下http -> server块内配置
   ```text
    server {
        #监听端口为 80
        listen       80;
        #设置主机域名
        server_name  www.xiaoyuge.work;
        #设置访问的语言编码
        #charset koi8-r;
        #设置虚拟主机访问日志的存放路径及日志的格式为main
        #access_log  logs/host.access.log  main;
   
        #    这个是域名反问的虚拟主机的文件路径
        root  /usr/local/nginx/data/ygb
        #设置虚拟主机的基本信息
        location / {
            #设置虚拟主机默认访问的网页
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
   ```

3. 启动,然后在浏览器访问域名www.xiaoyuge.work
    ```shell
      ./nginx -c ./nginx.conf
   ```