---
title: 服务器部署 VitePress
tags: [Linux, 建站]
categories: [服务器]
date: 2024-10-24
description: 鸢飞戾天，鱼跃于渊. 岂弟君子，遐不作人
references:
  - title: 部署 VitePress 站点 ( vitepress.dev )
    url: https://vitepress.dev/zh/guide/deploy#nginx
  - title: 基于vitepress搭建静态博客 ( llp-blog )
    url: https://blog.llinp.cn/article/hello.html
---

<br/>

## 基本环境

- node.js v18 及以上版本
- pnpm ( 可选, 以下基于 pnpm )

<br/>

```shell
pnpm add -D vitepress
```

<br/>


## 静态文件生成

```shell
git clone https://github.com/Daleveral/docs.git  

pnpm install

pnpm build  # alias pb in my ~/.zshrc
```

<br/>

等待 **~/docs/.vitepress/** 下生成存储静态网页的目录 **/dist**

```shell
cp -r ~/docs/.vitepress/dist/* /usr/share/nginx/html/dist
```

<br/>

## Nginx 配置
```shell
    server # notes  -->  https
    {
        listen 80;
        server_name notes.dalechu.cn;
        return 301 https://$server_name$request_uri;
    }


    server # notes  -->  /usr/share/nginx/html/dist
    {
        gzip on;
        gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        server_name notes.dalechu.cn;
        listen 443 ssl http2;

        ssl_stapling on;
        ssl_stapling_verify on;
        ssl_certificate notes.dalechu.cn.pem;
        ssl_certificate_key notes.dalechu.cn.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_ciphers "EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5";
        ssl_session_cache builtin:1000 shared:SSL:10m;
            
        location /  # 映射 dist 文件
        {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $http_host;
            root   /usr/share/nginx/html/dist;
            # index  index.html index.htm;
            try_files $uri $uri.html $uri/ =404;
            error_page 404 /404.html;
            error_page 403 /404.html;
            location ~* ^/assets/ 
            {
                expires 1y;
                add_header Cache-Control "public, immutable";
            }
        }
    }
```

<br/>

重启 Nginx

```shell
nginx -t
nginx -s reload
```

<br/>
