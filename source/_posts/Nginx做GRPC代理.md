---
title: Nginx做GRPC代理
date: 2023-11-13 15:07:56
tags:
categories:
- Go微服务组件
description: 详情点击阅读全文
---

## 明文传输
```conf
upstream backend {
    #  把下面的服务端地址和端口改成你自己的
    server 127.0.0.1:50052;
    server 127.0.0.1:50051;
}

server{
    listen 80 http2;

    location /hello.Hello {
        grpc_pass grpc://backend;
    }
}
```

## 加密传输
```conf 
upstream backend {
    #  把下面的服务端地址和端口改成你自己的
    server 127.0.0.1:50052;
    server 127.0.0.1:50051;
}

server{
    listen 80 default ssl http2;

    #证书(公钥.发送到客户端的)
    ssl_certificate ssl/server.crt;

    #私钥,
    ssl_certificate_key ssl/server.key;

    location /hello.Hello {
        grpc_pass grpcs://backend;
    }
}
```

## 已实验
```conf
upstream grpcservers {
  server 192.168.255.10:50051;
}

log_format  main    '$remote_addr - $remote_user [$time_local] "$request" '
           '$status $body_bytes_sent "$http_referer" '
           '"$http_user_agent"';

server {
        listen  80;
        server_name  haha.com;
        root   "D:/phpstudy_pro/WWW/haha.com";
        access_log  logs/access_grpc.log main;

        location / {

            # 重点！！需要将Content-Type更改为 application/grpc
            # grpc-web过来的是application/grpc-web+proto || application/grpc-web+text (取决于生成js代码时grpc-web_out 的mode选项，本文用grpcweb 则为application/grpc-web+proto)
            # grpc_set_header Content-Type application/grpc;
            # grpc_pass grpc://192.168.255.10:50051;

             # 因浏览器有跨域限制，这里直接在nginx支持跨域
            if ($request_method = 'OPTIONS') {
                add_header 'Access-Control-Allow-Origin' '*';
                add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
                add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Transfer-Encoding,Custom-Header-1,X-Accept-Content-Transfer-Encoding,X-Accept-Response-Streaming,X-User-Agent,X-Grpc-Web';
                add_header 'Access-Control-Max-Age' 1728000;
                add_header 'Content-Type' 'text/plain charset=UTF-8';
                add_header 'Content-Length' 0;
                return 204;
            }

            if ($request_method = 'POST') {
                add_header 'Access-Control-Allow-Origin' '*';
                add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
                add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Transfer-Encoding,Custom-Header-1,X-Accept-Content-Transfer-Encoding,X-Accept-Response-Streaming,X-User-Agent,X-Grpc-Web';
                add_header 'Access-Control-Expose-Headers' 'Content-Transfer-Encoding, grpc-message,grpc-status';
            }

            grpc_set_header Content-Type application/grpc;
            grpc_pass 192.168.255.10:50051;
            error_page 502 = /error502grpc;
        }

        location = /error502grpc {
            internal;
            default_type application/grpc;
            add_header grpc-status 14;
            add_header grpc-message "unavailable";
            return 204;
        }
}
```