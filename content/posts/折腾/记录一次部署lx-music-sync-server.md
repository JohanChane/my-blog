+++
title = "记录一次部署lx-music-sync-server"
date = "2025-10-14T00:00:00+08:00"
categories = ["折腾"]
tags = ["debian", "linux", "nginx", "反向代理"]
+++

## asdf 安装相应版本 go, node

安装 asdf 之后, 安装 go node

```sh
asdf plugin list all
asdf plugin add node

asdf list all nodejs
asdf install nodejs 16.20.2
```

## 编译 lx-music-sync-server 并测试

```sh
git clone https://github.com/lyswhut/lx-music-sync-server.git --depth 1
cd lx-music-sync-server

asdf set nodejs 16.20.2
asdf current

npm ci
npm run build
```

### 测试

config.js:

```js
'proxy.header': '<real ip>', // 代理转发的请求头，原始 IP

'env.PORT': '9527',
'env.BIND_IP': '0.0.0.0',
```

防火墙放开端口:

```sh
ufw allow 9527
```

另外的机器测试:

```sh
nmap <real ip>
curl http://<real ip>:9527/hello
```

有以下信息表示成功了:

```
Hello~::^-^::~v4~%
```

### 添加用户

在 config.js 的 `users` 下添加即可, 每个用户的密码不能相同, 因为以此区分不同的用户。

## 安装反向代理

### 关闭之前打开外网的东西

config.js:

```js
//'env.PORT': '9527',
//'env.BIND_IP': '0.0.0.0',

'proxy.enabled': true,
```

```sh
sudo ufw delete allow 9527
```

### 申请证书并安装证书

使用文章 "记录一次将宝塔的网站换成手工搭建" 的证书申请和安装脚本即可。比如我申请了 `lx.domain.com`。

### cloudflare 添加一条 `A` 记录

记得将该站点的流量代理打开。

### nginx

nvim /etc/nginx/sites-available/lx.domain.com:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name lx.domain.com;

    # 重定向 HTTP 到 HTTPS
    return 301 https://$server_name$request_uri;
}

map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name lx.domain.com;

    # SSL 证书路径
    ssl_certificate /etc/nginx/ssl/lx.domain.com/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/lx.domain.com/private.key;

    # SSL 配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    # HSTS (可选)
    add_header Strict-Transport-Security "max-age=63072000" always;

    location / {
        proxy_set_header X-Real-IP $remote_addr;  # 该头部与config.js文件的 proxy.header 对应
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host  $http_host;
        proxy_pass http://127.0.0.1:9527;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        # 代理超时设置
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        # 缓冲区设置
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
    }

    access_log /var/log/nginx/lx.domain.com.access.log;
    error_log /var/log/nginx/lx.domain.com.error.log;
}
```

```sh
ln -sf /etc/nginx/sites-available/lx.johandesigns.life /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

测试:

```sh
dig lx.domain.com # 查看 ip 有没有隐藏, 如果没有则在 cloudflare 将流量代理打开。
curl https://lx.domain.com/hello
```
