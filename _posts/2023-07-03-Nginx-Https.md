---
layout: post
title: '【网站快速部署】Nginx配置 + HTTPS证书申请'
subtitle: '在Ubuntu服务器上快速部署网站应用'
date: 2023.7.3
author: HenryHuang
tags:
    - Nginx
    - Certbot
    - ACME
    - HTTPS
---

当我们想配置一台服务器，用于部署网站。我们的需求是可以通过HTTPS网址直接访问，仅配置反向代理不能解决HTTPS证书的问题。接下来跟着如下教程步骤可省时省力快速完整部署：

***准备环境***

1. 新建Ubuntu服务器：

    磁盘镜像：Ubuntu
    公网IP：x.x.x.x
    防火墙：开启80，433端口访问

2. 域名服务商：

    创建域名解析：test.xxxx.com

3. 准备网站程序：

    在本地运行，部署一个可以本地发布的网站程序
    
    假设端口为8555
    
    ```
    http://localhost:8555
    ```


***开始部署***


1. 更新环境
	

	```
    sudo apt update
    sudo apt upgrade
	```

2. 安装所需软件：

    Nginx，certbot，python3-certbot-nginx

	```
    sudo apt install nginx
    sudo apt install python3-certbot-nginx
    sudo apt install certbot
	```

3. 配置Nginx：

    ```
    sudo vim /etc/nginx/sites-available/test.xxxx.com
	```

    进入VIM界面后，添加：

    ```
    server {
        listen 80;
        listen [::]:80;

        server_name test.xxxx.com;
        proxy_connect_timeout 20s;
        location / {
                proxy_pass http://127.0.0.1:8555/;
        }
    }
	```

    如果有配置WebSocket需求：

    ```
    server {
        listen 80;
        listen [::]:80;

        server_name test.xxxx.com;
        proxy_connect_timeout 20s;
        location / {
                proxy_pass http://127.0.0.1:8555/;
                proxy_redirect off;
                proxy_http_version 1.1;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $host;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_read_timeout 86400;
        }
    }
	```

4. 检查配置生效：

    将sites-available内的配置文件设置路径到site-enabled

    ```
    sudo ln -s /etc/nginx/sites-available/test.xxxx.com /etc/nginx/sites-enabled/
	```
    
    检查配置

    ```
    sudo nginx -t
	```

    没有报错则重新加载Nginx

    ```
    sudo systemctl reload nginx
	```


## 方案一 使用Certbot

Certbot可以有效帮助你管理证书，并自动帮你更新。最优解决方案

    
5. 配置Certbot管理证书：

    配置Certbot
    
    注意第一次配置需要选择是否登记邮箱，用于接收重要的通知，例如证书即将过期的警告。

    如果不想登记，在一个选项的时候选：c （Cancel）

    ```
    sudo certbot --nginx -d test.xxxx.com
    ```

    检查是否生效：

    ```
    cat /etc/nginx/sites-available/test.xxxx.com
    ```

    如果生效，打印出来的结果应包含443端口监听：

    ```
    server {
        listen 80;
        listen [::]:80;

        server_name test.xxxx.com;
        proxy_connect_timeout 20s;
        location / {
                proxy_pass http://127.0.0.1:8555/;
            }

        listen 443 ssl; # managed by Certbot
        ssl_certificate /etc/letsencrypt/live/test.xxxx.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/test.xxxx.com/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    }
    ``` 

6. 配置网站运行环境：

    剩下步骤按照本地可运行环境进行配置即可，注意端口为8555，与Nginx配置对应


## 方案二 使用ACME申请

当使用Certbot运行时报错时，可以选择使用acme.sh脚本申请

这是一个纯Shell脚本，用于从Let's Encrypt或其他ACME协议的CA获取SSL证书。

后续步骤涉及当前用户目录，假设当前用户为testuser，用户目录为/home/testuser

从前序第四步之后继续：

5. 安装'acme.sh':

    ``` 
    curl https://get.acme.sh | sh
    ``` 
6. 注册账户：

    使用电子邮件地址注册一个账户。这是一个必须步骤，这些证书管理机构需要一个电子邮件地址来发送重要的通知，例如证书即将过期的警告。

    ```
    /home/testuser/.acme.sh/acme.sh --register-account -m your_email@example.com
    ```

7. 获取证书：

    ```
    /home/testuser/.acme.sh/acme.sh --issue --nginx -d test.xxxx.com
    ```

    如果在这一步显示你当前用户权限不够，可使用sudo命令，或直接使用root用户进行后续操作

8. 安装证书：

    假设你希望将证书存储在 /etc/nginx/ssl 目录下

    创建/etc/nginx/ssl 目录

    ```
    sudo mkdir -p /etc/nginx/ssl
    ```

    安装证书

    ```
    /home/testuser/.acme.sh/acme.sh --install-cert -d test.xxxx.com \
    --key-file       /etc/nginx/ssl/key.pem  \
    --fullchain-file /etc/nginx/ssl/cert.pem \
    --reloadcmd     "service nginx force-reload"
    ```

    当前生成的证书，在 

    ```
    /root/.acme.sh/test.xxxx.com 
    ```

    或者

    ```
    /root/.acme.sh/test.xxxx.com_ecc
    ```
    
    目录下

    证书文件包括：

    - test.xxxx.com.cer: 这是服务器证书，也就是你的域名证书
    - ctest.xxxx.com.key: 这是你的私钥
    - ca.cer: 这是中间证书
    - fullchain.cer: 这是完整链证书（包含服务器证书和中间证书）

9. 配置Nginx使HTTPS证书生效：

    对应Certbot配置，还需要一个Diffie-Hellman参数文件

    ```
    openssl dhparam -out /etc/nginx/ssl-dhparams.pem 2048
    ```

    重新打开Nginx配置文件：

    ```
    sudo vim /etc/nginx/sites-available/test.xxxx.com
	```

    在location字段后手动加上：

    ```
    listen 443 ssl;
    ssl_certificate /root/.acme.sh/test.xxxx.com_ecc/fullchain.cer;
    ssl_certificate_key /root/.acme.sh/test.xxxx.com_ecc/test.xxxx.com.key;
    ssl_dhparam /etc/nginx/ssl-dhparams.pem;
    ```

    添加后完整的文件内容如下：

    ```
    server {
        listen 80;
        listen [::]:80;

        server_name test.xxxx.com;
        proxy_connect_timeout 20s;
        location / {
                proxy_pass http://127.0.0.1:8555/;
            }

        listen 443 ssl;
        ssl_certificate /root/.acme.sh/test.xxxx.com_ecc/fullchain.cer;
        ssl_certificate_key /root/.acme.sh/test.xxxx.com_ecc/test.xxxx.com.key;
        ssl_dhparam /etc/nginx/ssl-dhparams.pem;

    }
    ``` 

    检查配置

    ```
    sudo nginx -t
	```

    没有报错则重新加载Nginx使之生效

    ```
    sudo systemctl reload nginx
	```

10. 配置网站运行环境：

    剩下步骤按照本地可运行环境进行配置即可，注意端口为8555，与Nginx配置对应


***由此可直接使用HTTPS网址进行访问***