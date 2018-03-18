---
title: 全站启用 HTTPS，Nginx 反向代理部署 Angular 5 Web App
categories: 程序猿
tags:
  - HTTPS
  - HTTP/2
  - Nginx
  - Angular
  - SSL
  - GitHub Pages
  - VPS
  - 网站开发
abbrlink: 3cd67903
date: 2018-03-17 15:42:54
---

其实从好几年之前整个互联网就开始将 WEB 服务从 HTTP 迁移到 HTTPS，而现在为了更快的推进 HTTPS 的普及，Chrome 将从 `2018 年 7 月`起标记所有的 HTTP 网站为`不安全`链接。从这一点来说，我是支持的，HTTPS 会逐渐成为 WEB 服务的标配，最最重要的是，它能给用户带来更安全、更好隐私保护的网络体验。具体来说，防止网站数据在网络传输中被窃听和篡改。一个典型的例子就是国内的宽带运营商劫持，强行在用户访问网站的时候嵌入广告、修改页面、甚至强行跳转。而一些 HTML5 API 可以用来获取用户地理位置、摄像头音视频等隐私数据。尽管网站在使用这些权限前需要获得用户授权，但由于 HTTP 很容易被劫持，通过 HTTP 传输这些隐私数据将会十分危险。因此，我建议大家尽早升级，现在通过 Let's Encrypt 获取免费证书很容易，虽然证书只有 90 天有效期，只需要一个简单的脚本就能自动更新。

这篇文章主要记录自己的[博客站](https://kris2d.info/)以及运行在 VPS 上的 [Angular 5 Web App](https://crypto.kris2d.info/) 启用 HTTPS 的过程和一些经验，分享给大家。

![https-cover](https://user-images.githubusercontent.com/5259084/37552677-0e069a02-2a0e-11e8-8eff-81d9ca900371.jpg)

<!--more-->

------

### 我的博客

如果博客构建在 GitHub Pages，启用 HTTPS 分两种情况：

* 无自定义域名，那么 GitHub 已经支持默认域名 `*.github.io` 启用强制 HTTPS 连接，在 repo 的设置中就可以打开；
* 自定义域名，例如我的博客域名为 `kris2d.info`，需借助 [Cloudflare](https://www.cloudflare.com/) 提供的免费 CDN 服务，为博客开启 HTTPS 支持，同时也提供 HTTP/2 的支持。

#### 设置 Nameserver

将域名的 Nameserver 指向 Cloudflare 提供给你的 Nameserver，例如我的设置如下（子域名 crypto 是我的 Web App 在使用）：

![https-cloudflare-dns](https://user-images.githubusercontent.com/5259084/37552886-7ccecb1e-2a11-11e8-85e4-7f7513463070.jpg)

#### 开启 SSL

在 Cloudflare 的站点管理页面，切换到 Crypto 标签页，将 SSL 的模式设置为 `Full`，如下图所示：

![https-cloudflare-ssl](https://user-images.githubusercontent.com/5259084/37552997-63f3a1da-2a13-11e8-8f98-4dda6edd1305.png)

注意不能设为 Full (strict)，现在该域名就可以支持 HTTPS 访问了。但是你会发现，如果直接输入域名，浏览器仍然默认以 HTTP 协议来访问，并不会自动跳转到 HTTPS。

#### HTTPS 跳转

Cloudflare 提供一个名叫 Page Rules 页面规则的功能，可以匹配不同规则的 URL 做一些处理。切换至 Pages Rules 标签页，新建如下规则：

![https-cloudflare-pagerules](https://user-images.githubusercontent.com/5259084/37553047-314c4452-2a14-11e8-8b88-db5af3e05783.png)

完成后点击 `Save and Deploy`，至此，博客站就实现了全站 HTTPS 安全连接。需要提醒的是，博客中如果使用外链的图片，也必须为 HTTPS 链接，否则 Chrome 等浏览器会有 `Mixed Content` 错误提示。而一些社区分享插件也不支持 HTTPS，同样需要禁用才能消除错误提示。

### Angular 5 Web App

我的这个 App 算是练手用项目，就没有再申请新的域名，使用的是子域名 `crypto.kris2d.info`。自己有一个 VPS，运行 `Ubuntu Server 17.10`，搭建环境为 [http-server](https://www.npmjs.com/package/http-server)、[PM2](http://pm2.keymetrics.io/)、以及 [Nginx](https://www.nginx.com/) 作反向代理。申请 [Let's Encrypt](https://letsencrypt.org/) 使用 [Certbot](https://certbot.eff.org/)，根据提示就可以获得免费的 SSL 证书，添加到 Nginx Site Conf 中就可以了。

#### 先决条件

* Ubuntu 中安装 `node.js` 和 `npm`

```bash
sudo apt-get update
sudo apt-get install nodejs
sudo apt-get install npm
```

* 安装 `http-server` 和 `PM2`

```bash
npm install http-server -g
npm install pm2 -g
```

这里，我们就可以在任意目录创建一个 PM2 可以读取的 json 文件，用于 http-server 的不间断运行。这里我设置静态服务器的监听端口为 3000，文件另存为 procee.json：

```json
{
    "apps": [
        {
            "name": "angular",
            "cwd": "/var/www/crypto.kris2d.info/dist",
            "args": "-p 3000 -d false",
            "script": "/usr/local/lib/node_modules/http-server/bin/http-server"
        }
    ]
}
```

之后，只要运行 `pm2 start /path/to/process.json`，Angular App 就开始正常运行了。

#### 编译 Nginx

我没有直接使用 Ubuntu 系统库中的 Nginx，而是使用 [nginx-autoinstall](https://github.com/Angristan/nginx-autoinstall) 来安装附带额外功能的版本，例如通过编译新版本的 OpenSSL 从而支持 HTTP/2、ChaCha20 cipher 等，同时打上了 Cloudflare's TLS Dynamic Records Resizing patch 补丁。

```bash
wget https://raw.githubusercontent.com/Angristan/nginx-autoinstall/master/nginx-autoinstall.sh
chmod +x nginx-autoinstall.sh
./nginx-autoinstall.sh
```

按照提示，分别选择 `Stable` 版本，Modules 中选择了 `Brotli`、`GeoIP` 和 `Cloudflare's TLS Dynamic Records Resizing patch`，在 OpenSSL 中选择 `OpenSSL from source` 等待完成即可。最后添加基本的 Nginx 站点配置文件，例如我的站点，增加一个配置文件，位于 `/etc/nginx/sites-available/crypto.kris2d.info`，并且创建一个 symbolic link 到 `/etc/nginx/sites-enabled/crypto.kris2d.info`。

#### 获取 Let's Encrypt 证书

* 安装 Certbot

```bash
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install python-certbot-nginx 
```

* 获取当前域名的免费 SSL 证书

```bash
sudo certbot --nginx -d crypto.kris2d.info
```

按照提示同意条款并输入邮箱地址，最后的提示 Redirect 自动配置可以随意选择，之后会修改。完成后，记录好证书和 conf 文件的位置，默认情况下，证书位于 `/etc/letsencrypt/` 目录中。

### Nginx 反向代理部署 Angular 5

目前 App 是运行在 http-server 上的，由 Nginx 作反向代理，完整的站点配置如下：

```nginx
upstream node_server {
        server 127.0.0.1:3000;
}

server {
        listen 80;

        server_name crypto.kris2d.info;
        root /var/www/crypto.kris2d.info/dist;

        if ($request_method !~ ^(GET|HEAD|POST)$ ) {
                return 444;
        }

        location / {
                rewrite ^/(.*)$ https://crypto.kris2d.info/$1 permanent;
        }
}

server {
        listen 443 ssl http2 fastopen=3 reuseport;
        server_name crypto.kris2d.info;
        server_tokens off;

        root /var/www/crypto.kris2d.info/dist;

        ssl_certificate /etc/letsencrypt/live/crypto.kris2d.info/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/crypto.kris2d.info/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        ssl_session_tickets on;

        ssl_stapling on;
        ssl_stapling_verify on;
        resolver 8.8.4.4 8.8.8.8 valid=300s;
        resolver_timeout 10s;

        if ($request_method !~ ^(GET|HEAD|POST|OPTIONS)$ ) {
                return 444;
        }

        location ~ ^/(scripts.*js|styles|images) {
                expires 4h;
                add_header Cache-Control public;
                add_header ETag "";

                break;
        }

        location / {
                proxy_http_version 1.1;

                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-NginX-Proxy true;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";

                proxy_cache_bypass $http_upgrade;

                add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
                add_header X-Frame-Options deny;
                add_header X-Content-Type-Options nosniff;

                proxy_hide_header Vary;
                proxy_hide_header X-Powered-By;

                proxy_redirect off;
                proxy_pass http://node_server;
                try_files $uri $uri/ /index.html;
        }
}
```

其中，对 SSL 证书的配置文件内容稍作修改：

```nginx
ssl_session_cache shared:le_nginx_SSL:50m;
ssl_session_timeout 1d;

ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;

ssl_ciphers "ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE$
```

配置完成后，运行 `nginx -t` 确认配置文件无误，运行 `systemctl restart nginx` 就可以加载新的配置。此时，我的 Web App 就完全支持 HTTPS 和 HTTP/2 了。需要注意的是，上述部分配置与 Nginx 性能和安全优化有关，应根据自己的情况进行修改，具体含义可以访问 [Nginx Doc](https://nginx.org/en/docs/) ，如果遇到问题也可以给我留言。

一切配置妥当，最终我在 [Qualys SSL Labs SSL Server Test](https://www.ssllabs.com/ssltest/) 的测试结果达到 A+，测试结果如下图：

![https-server-test](https://user-images.githubusercontent.com/5259084/37555243-8dc99f12-2a38-11e8-9732-16079629f619.png)

使用 Nginx 直接走 HTTP 代理部署 Angular 5 App 很简单，而使用其进行反向代理在配置的时候遇到了很多问题，最终查阅了不少资料才找到解决方法。希望我的这篇文章对大家有所帮助，那么下次见，Peace！
