---
title: 基于 frp 的教育网反向代理
tags:
  - frp
  - Nginx
---

# 背景

在教育网内有一台服务器提供 WEB 服务（假设其教育网 IP 是 `202.38.64.246`），绑定域名 `example.com`。教育网 IP 的 443 端口在公网无法访问，但有一台公网服务器（假设 IP 是 `112.80.248.75`）。

现希望在教育网内访问 `https://example.com` 网站时能直达教育网服务器，在公网访问 `https://example.com` 能反向代理到教育网服务器：即用户可以不关心自己的网络情况透明地访问 `https://example.com`。

为了让问题一般化，假设公网服务器上原先还在 443 端口跑着 `https://example.org` 网站，我们希望反向代理 `https://example.com` 不影响 `https://example.org` 的正常运行。



# 方案

## 基本思路

借助 DNS 解析服务商的“智能解析”，在教育网线路解析到教育网真实 IP，在公网线路解析到公网服务器 IP，同时这个公网服务器运行 frp server 端，教育网服务器运行 frp client 端，公网服务器借助 Nginx 和 frp 将请求反向代理至教育网服务器（实际上这里的 Nginx 和 frp 各自运行了一个独立的反向代理服务，即串联了两层反向代理）。


![](https://img.yusanshi.com/upload/20220704121229461491.png)



## 具体过程

### 教育网服务器

#### frp

**frpc.ini**

使公网服务器 `112.80.248.75` 的 10443 端口为本地 443 端口提供反向代理。

```ini
[common]
server_addr = 112.80.248.75
server_port = 7000
authentication_method = token
authenticate_new_work_conns = true
token = here_is_the_token

[tcp_port]
type = tcp
local_ip = 127.0.0.1
local_port = 443
remote_port = 10443
```

**frpc.service**（用于开机自启）

```ini
[Unit]
Description=Frp Client Service
After=network.target

[Service]
Type=simple
User=nobody
Restart=on-failure
RestartSec=5s
ExecStart=/usr/bin/frpc -c /etc/frp/frpc.ini
ExecReload=/usr/bin/frpc reload -c /etc/frp/frpc.ini
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```



#### Nginx

保持原始正常配置即可，如：

```nginx
server {
    listen 80 default_server;
    server_name _;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name example.com;
    root ...;
    index index.html;
    ssl_certificate ...;
    ssl_certificate_key ...;
    location / {
        try_files $uri $uri/ =404;
    }
}
```



### 公网服务器

#### 防火墙放行

在服务器控制面板放行 frp 的 7000 端口。

![](https://img.yusanshi.com/upload/20211108211019688737.png)

#### frp

**frps.ini**

```ini
[common]
bind_port = 7000
authentication_method = token
authenticate_new_work_conns = true
token = here_is_the_token
```

**frps.service**（用于开机自启）

```ini
[Unit]
Description=Frp Server Service
After=network.target

[Service]
Type=simple
User=nobody
Restart=on-failure
RestartSec=5s
ExecStart=/usr/bin/frps -c /etc/frp/frps.ini
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
```

#### Nginx

它的作用是将访问 `https://example.com` 的请求反向代理至本地 10443 端口。这里提供两种思路：使用 Nginx `stream` 块实现的 SSL Passthrough 和使用 Nginx `http` 块实现的 SSL Bridging。

它们的简单对比如下表：

|                                   | SSL Passthrough                             | SSL Bridging                               |
| --------------------------------- | ------------------------------------------- | ------------------------------------------ |
| 速度                              | Good（反向代理服务器不解密流量）            | Bad（反向代理服务器解密、再加密）          |
| 对原网站 `example.org` 配置的影响 | Bad（需要改原网站的监听端口）               | Good（不用改原网站的配置）                 |
| 反向代理服务器证书配置            | Good（反向代理服务器上不需要提供 SSL 证书） | Bad（反向代理服务器上也需要提供 SSL 证书） |
| 反向代理服务器端流量控制能力      | Bad（流量未解密，不能为所欲为）             | Good（流量解密后可以为所欲为）             |
| 安全性                            | Good                                        | Good                                       |

##### SSL Passthrough

下图偷自 <https://parksehun.medium.com/ssl-passthrough-vs-ssl-termination-vs-ssl-bridging-f66b24d4d0aa>。

![](https://img.yusanshi.com/upload/20211108220559145547.png)

```nginx
http {
    server {
        listen 80 default_server;
        server_name _ ;
        return 301 https://$host$request_uri;
    }

    server {
        listen 9443 ssl; # 注意这里需要把 443 改成别的端口，因为 stream 块中已经在监听 443 端口了
        server_name example.org;
        ...
    }
}

stream {
    map $ssl_preread_server_name $name {
        example.com edu_backend; # 代理流量导向本地 frps serve 的 10443 端口
        default local_backend; # 其他流量导向本地 9443 端口
    }

    upstream edu_backend {
        server 127.0.0.1:10443;
    }

    upstream local_backend {
        server 127.0.0.1:9443;
    }

    server {
        listen 443;
        proxy_pass $name;
        ssl_preread on;
    }
}

```

##### SSL Bridging

下图图源和上图相同。

![](https://img.yusanshi.com/upload/20211108220519146044.png)

```nginx
http {
    server {
        listen 80 default_server;
        server_name _ ;
        return 301 https://$host$request_uri;
    }

    # example.org 的配置保持不变
    server {
        listen 443 ssl;
        server_name example.org;
        ...
    }

    server {
        listen 443 ssl;
        server_name example.com;
        # 这里仍需要 example.com 的证书
        ssl_certificate ...;
        ssl_certificate_key ...;
        location / {
            proxy_pass https://127.0.0.1:10443;
            proxy_set_header    Host            $host;
            proxy_set_header    X-Real-IP       $remote_addr;
            proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}

stream {
    # 不需要这个块
}
```

### 域名解析

借助 DNS 解析服务商的“智能解析”，在教育网线路解析到教育网真实 IP，在公网线路解析到公网服务器 IP。

![](https://img.yusanshi.com/upload/20211108221305937566.png)