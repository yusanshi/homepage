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


<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" viewBox="-0.5 -0.5 912 615" content="&lt;mxfile host=&quot;app.diagrams.net&quot; modified=&quot;2021-11-08T15:10:25.235Z&quot; agent=&quot;5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/95.0.4638.69 Safari/537.36&quot; etag=&quot;TiNuePOJMeFxDscjq6MA&quot; version=&quot;15.3.0&quot; type=&quot;google&quot;&gt;&lt;diagram id=&quot;qGVKnMUrzwl4tf_F5i0A&quot; name=&quot;Page-1&quot;&gt;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&lt;/diagram&gt;&lt;/mxfile&gt;" style="background-color: rgb(255, 255, 255);"><defs/><g><rect x="30" y="30" width="290" height="210" fill-opacity="0.6" fill="#d5e8d4" stroke="#82b366" stroke-opacity="0.6" pointer-events="all"/><rect x="500" y="30" width="270" height="210" fill-opacity="0.6" fill="#d5e8d4" stroke="#82b366" stroke-opacity="0.6" pointer-events="all"/><path d="M 360 480 Q 350 390 280 360 Q 210 330 191.58 247.13" fill="none" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 190.36 241.64 L 195.65 248.14 L 191.58 247.13 L 188.33 249.77 Z" fill="#000000" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="all"/><path d="M 360 560 L 360 480" fill="none" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 360 480 Q 370 350 480 330 Q 590 310 614.83 246.8" fill="none" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 616.89 241.56 L 617.63 249.91 L 614.83 246.8 L 610.65 247.17 Z" fill="#000000" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="all"/><rect x="240" y="570" width="232.5" height="10" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 231px; height: 1px; padding-top: 575px; margin-left: 241px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 16px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">GET https://example.com</div></div></div></foreignObject><text x="356" y="580" fill="#000000" font-family="Helvetica" font-size="16px" text-anchor="middle">GET https://example.com</text></switch></g><rect x="322" y="481" width="75" height="10" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 73px; height: 1px; padding-top: 486px; margin-left: 323px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 16px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">智能解析</div></div></div></foreignObject><text x="360" y="491" fill="#000000" font-family="Helvetica" font-size="16px" text-anchor="middle">智能解析</text></switch></g><rect x="235" y="420" width="110" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 430px; margin-left: 290px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 16px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: nowrap;">如果是教育网</div></div></div></foreignObject><text x="290" y="435" fill="#000000" font-family="Helvetica" font-size="16px" text-anchor="middle">如果是教育网</text></switch></g><rect x="370" y="420" width="50" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 1px; height: 1px; padding-top: 430px; margin-left: 395px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 16px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: nowrap;">其他</div></div></div></foreignObject><text x="395" y="435" fill="#000000" font-family="Helvetica" font-size="16px" text-anchor="middle">其他</text></switch></g><rect x="150" y="180" width="80" height="60" fill-opacity="0.7" fill="#ffcc99" stroke="#36393d" stroke-opacity="0.7" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 78px; height: 1px; padding-top: 210px; margin-left: 151px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 16px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">Nginx</div></div></div></foreignObject><text x="190" y="215" fill="#000000" font-family="Helvetica" font-size="16px" text-anchor="middle">Nginx</text></switch></g><rect x="595" y="180" width="90" height="60" fill-opacity="0.7" fill="#ffcc99" stroke="#36393d" stroke-opacity="0.7" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 88px; height: 1px; padding-top: 210px; margin-left: 596px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 16px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">Nginx</div></div></div></foreignObject><text x="640" y="215" fill="#000000" font-family="Helvetica" font-size="16px" text-anchor="middle">Nginx</text></switch></g><rect x="260" y="70" width="60" height="50" fill-opacity="0.7" fill="#ffcc99" stroke="#36393d" stroke-opacity="0.7" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 58px; height: 1px; padding-top: 95px; margin-left: 261px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 16px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">frpc</div></div></div></foreignObject><text x="290" y="100" fill="#000000" font-family="Helvetica" font-size="16px" text-anchor="middle">frpc</text></switch></g><rect x="500" y="70" width="60" height="50" fill-opacity="0.7" fill="#ffcc99" stroke="#36393d" stroke-opacity="0.7" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 58px; height: 1px; padding-top: 95px; margin-left: 501px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 16px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">frps</div></div></div></foreignObject><text x="530" y="100" fill="#000000" font-family="Helvetica" font-size="16px" text-anchor="middle">frps</text></switch></g><path d="M 617.5 240 Q 618 150 565.3 100.02" fill="none" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 561.22 96.15 L 569.24 98.59 L 565.3 100.02 L 564.08 104.04 Z" fill="#000000" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="all"/><path d="M 260 95 Q 190 110 190 172.7" fill="none" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 190 178.32 L 186.25 170.82 L 190 172.7 L 193.75 170.82 Z" fill="#000000" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="all"/><rect x="590" y="570" width="220" height="10" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 218px; height: 1px; padding-top: 575px; margin-left: 591px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 16px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">GET https://example.org</div></div></div></foreignObject><text x="700" y="580" fill="#000000" font-family="Helvetica" font-size="16px" text-anchor="middle">GET https://example.org</text></switch></g><path d="M 680 560 Q 690 370 664.01 247.14" fill="none" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="stroke"/><path d="M 662.85 241.64 L 668.07 248.2 L 664.01 247.14 L 660.73 249.75 Z" fill="#000000" stroke="#000000" stroke-width="1.5" stroke-miterlimit="10" pointer-events="all"/><rect x="30" y="30" width="150" height="60" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 148px; height: 1px; padding-top: 60px; margin-left: 31px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 20px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;"><font style="font-size: 20px">教育网服务器</font></div></div></div></foreignObject><text x="105" y="66" fill="#000000" font-family="Helvetica" font-size="20px" text-anchor="middle">教育网服务器</text></switch></g><rect x="640" y="30" width="130" height="60" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 128px; height: 1px; padding-top: 60px; margin-left: 641px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 20px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">公网服务器</div></div></div></foreignObject><text x="705" y="66" fill="#000000" font-family="Helvetica" font-size="20px" text-anchor="middle">公网服务器</text></switch></g><rect x="690" y="380" width="190" height="50" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe flex-start; width: 188px; height: 1px; padding-top: 405px; margin-left: 692px;"><div style="box-sizing: border-box; font-size: 0px; text-align: left;"><div style="display: inline-block; font-size: 14px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;"><span style="font-size: 14px">DEST: 112.80.248.75:</span><span style="font-size: 14px">443<br style="font-size: 14px" /></span><span style="font-size: 14px">HOST: example.org:443</span></div></div></div></foreignObject><text x="692" y="409" fill="#000000" font-family="Helvetica" font-size="14px">DEST: 112.80.248.75:443...</text></switch></g><rect x="450" y="330" width="190" height="50" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe flex-start; width: 188px; height: 1px; padding-top: 355px; margin-left: 452px;"><div style="box-sizing: border-box; font-size: 0px; text-align: left;"><div style="display: inline-block; font-size: 14px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;"><span style="font-size: 14px">DEST: 112.80.248.75:</span><span style="font-size: 14px">443<br style="font-size: 14px" /></span><span style="font-size: 14px">HOST: example.com:443</span></div></div></div></foreignObject><text x="452" y="359" fill="#000000" font-family="Helvetica" font-size="14px">DEST: 112.80.248.75:443...</text></switch></g><rect x="60" y="310" width="190" height="50" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe flex-start; width: 188px; height: 1px; padding-top: 335px; margin-left: 62px;"><div style="box-sizing: border-box; font-size: 0px; text-align: left;"><div style="display: inline-block; font-size: 14px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;"><span style="font-size: 14px">DEST: 202.38.64.246:</span><span style="font-size: 14px">443<br style="font-size: 14px" /></span><span style="font-size: 14px">HOST: example.com:443</span></div></div></div></foreignObject><text x="62" y="339" fill="#000000" font-family="Helvetica" font-size="14px">DEST: 202.38.64.246:443...</text></switch></g><rect x="545" y="120" width="190" height="50" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe flex-start; width: 188px; height: 1px; padding-top: 145px; margin-left: 547px;"><div style="box-sizing: border-box; font-size: 0px; text-align: left;"><div style="display: inline-block; font-size: 14px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;"><span style="font-size: 14px">DEST: 127.0.0.1:10</span><span style="font-size: 14px">443<br style="font-size: 14px" /></span><span style="font-size: 14px">HOST: example.com:443</span></div></div></div></foreignObject><text x="547" y="149" fill="#000000" font-family="Helvetica" font-size="14px">DEST: 127.0.0.1:10443...</text></switch></g><rect x="90" y="110" width="190" height="50" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe flex-start; width: 188px; height: 1px; padding-top: 135px; margin-left: 92px;"><div style="box-sizing: border-box; font-size: 0px; text-align: left;"><div style="display: inline-block; font-size: 14px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;"><span style="font-size: 14px">DEST: 127.0.0.1:</span><span style="font-size: 14px">443<br style="font-size: 14px" /></span><span style="font-size: 14px">HOST: example.com:443</span></div></div></div></foreignObject><text x="92" y="139" fill="#000000" font-family="Helvetica" font-size="14px">DEST: 127.0.0.1:443...</text></switch></g><path d="M 320 92.5 L 500 92.5 M 500 97.5 L 320 97.5 M 500 97.5" fill="none" stroke="#000000" stroke-width="2" stroke-linejoin="round" stroke-miterlimit="10" pointer-events="all"/><rect x="340" y="120" width="240" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe flex-start; width: 238px; height: 1px; padding-top: 130px; margin-left: 342px;"><div style="box-sizing: border-box; font-size: 0px; text-align: left;"><div style="display: inline-block; font-size: 14px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">The magic frp that maps<br /><span>112.80.248.75:10443<br /></span>to <span>202.38.64.246:</span><span>443</span></div></div></div></foreignObject><text x="342" y="134" fill="#000000" font-family="Helvetica" font-size="14px">The magic frp that maps...</text></switch></g></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"/><a transform="translate(0,-5)" xlink:href="https://www.diagrams.net/doc/faq/svg-export-text-problems" target="_blank"><text text-anchor="middle" font-size="10px" x="50%" y="100%">Viewer does not support full SVG 1.1</text></a></switch></svg>



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