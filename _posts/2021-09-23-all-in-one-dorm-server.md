---
title: All in One Dorm Server 配置记录
tags:
  - router
  - server
  - Nginx
  - Proxmox
  - OpenWrt
---

最近看服务器/软路由心潮澎湃，想买一台 HPE MicroServer Gen10 Plus，奈何手头有点紧张，一番思考之后，决定先买个更便宜的设备，等以后有了自己的房子再来上大家伙。于是就在淘宝选了个 J4125 的软路由。因为手头有两条镁光内存条（8GB DDR4 2400），就选了准系统版本并且额外买了个 mSATA 的固态硬盘（我手边有 M2 的固态奈何那个软路由只有 mSATA 接口）。



## 硬件

| 项目 | 型号                   |
| ---- | ---------------------- |
| CPU  | Intel Celeron J4125    |
| 内存 | 镁光 8GB DDR4 2400 x 2 |
| 硬盘 | 三星 850EVO 250G mSATA |
| 网卡 | Intel I210 x 4         |



**俯视图**

![](https://img.yusanshi.com/upload/20210923185410850842.jpg)



## 软件

**Host**

Proxmox VE 7.0-2

**Virtual Machines**

- OpenWrt 21.02.0 x86-64

- Ubuntu 20.04.3


![](https://img.yusanshi.com/upload/20220704120918310773.png)


## 配置文件

### Host

#### `/etc/network/interfaces`

这个配置文件里的 `eno1` 的别名是 `enp3s0`，但是我不知道怎么把它还原成 `enp3s0`（或者说怎么优雅地做这件事，我找到的方案很复杂）。

```
## /etc/network/interfaces
auto lo
iface lo inet loopback

iface enp1s0 inet manual

iface enp2s0 inet manual

iface eno1 inet manual

iface enp4s0 inet manual

auto vmbr0
iface vmbr0 inet manual
	bridge-ports enp1s0
	bridge-stp off
	bridge-fd 0

auto vmbr1
iface vmbr1 inet manual
	bridge-ports enp2s0
	bridge-stp off
	bridge-fd 0

auto vmbr2
iface vmbr2 inet manual
	bridge-ports eno1
	bridge-stp off
	bridge-fd 0

auto vmbr3
iface vmbr3 inet manual
	bridge-ports enp4s0
	bridge-stp off
	bridge-fd 0

```

#### `host.dorm.yusanshi.com.conf`

```
ssl_certificate /root/cert/fullchain.pem;
ssl_certificate_key /root/cert/privkey.pem;


### http -> https
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _ ;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    listen 8000 ssl default_server;
    listen [::]:8000 ssl default_server;
    server_name _;

    error_page 497 https://$host:$server_port$request_uri;

    return 301 https://host.dorm.yusanshi.com;
}

### host.dorm.yusanshi.com
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    listen 8000 ssl;
    listen [::]:8000 ssl;
    server_name host.dorm.yusanshi.com;

    error_page 497 https://$host:$server_port$request_uri;

    location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_pass https://127.0.0.1:8006;
        proxy_buffering off;
        client_max_body_size 0;
        proxy_connect_timeout  3600s;
        proxy_read_timeout  3600s;
        proxy_send_timeout  3600s;
        send_timeout  3600s;
    }
}

```



### Virtual Machines

#### OpenWrt

##### `/etc/config/network`

```
## /etc/config/network
config interface 'loopback'
	option device 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config globals 'globals'
	option ula_prefix 'fd45:3134:fcfc::/48'

config device
	option name 'br-lan'
	option type 'bridge'
	list ports 'eth1'
	list ports 'eth2'
	list ports 'eth3'

config interface 'lan'
	option device 'br-lan'
	option proto 'static'
	option ipaddr '192.168.1.1'
	option netmask '255.255.255.0'
	option ip6assign '60'

config interface 'wan'
	option proto 'dhcp'
	option device 'eth0'

config interface 'wan6'
	option proto 'dhcpv6'
	option device 'eth0'

```

##### `/etc/config/dhcp`

参考自 <https://openwrt.org/docs/guide-user/network/ipv6/start#ipv6_relay>，自己在原内容上加的只有：

```
config dhcp lan
    option dhcpv6 relay
    option ra relay
    option ndp relay
 
config dhcp wan6
    option dhcpv6 relay
    option ra relay
    option ndp relay
    option master 1
    option interface wan6
```

下面是添加了 IPv6 relay 配置后的文件：

```
## /etc/config/dhcp
config dnsmasq
	option domainneeded '1'
	option boguspriv '1'
	option filterwin2k '0'
	option localise_queries '1'
	option rebind_protection '1'
	option rebind_localhost '1'
	option local '/lan/'
	option domain 'lan'
	option expandhosts '1'
	option nonegcache '0'
	option authoritative '1'
	option readethers '1'
	option leasefile '/tmp/dhcp.leases'
	option nonwildcard '1'
	option localservice '1'
	option ednspacket_max '1232'
	option filter_aaaa '0'
	option resolvfile '/tmp/resolv.conf.d/resolv.conf.auto'
	option noresolv '0'

config dhcp 'lan'
	option interface 'lan'
	option start '100'
	option limit '150'
	option leasetime '12h'
	option dhcpv4 'server'
	option dhcpv6 'relay'
	option ra 'relay'
	option ndp 'relay'
	option ra_slaac '1'
	list ra_flags 'managed-config'
	list ra_flags 'other-config'

config dhcp 'wan'
	option interface 'wan'
	option ignore '1'

config dhcp 'wan6'
	option dhcpv6 'relay'
	option ra 'relay'
	option ndp 'relay'
	option master '1'
	option interface 'wan6'

config odhcpd 'odhcpd'
	option maindhcp '0'
	option leasefile '/tmp/hosts/odhcpd'
	option leasetrigger '/usr/sbin/odhcpd-update'
	option loglevel '4'

```

#### Ubuntu

##### `/etc/netplan/00-installer-config.yaml`

```
## /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    ens18:
      dhcp4: true
      dhcp6: true
  version: 2
```

##### `dorm.yusanshi.com.conf`

```
ssl_certificate /home/yu/cert/fullchain.pem;
ssl_certificate_key /home/yu/cert/privkey.pem;

### http -> https
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _ ;
    return 301 https://$host$request_uri;
}

### *.dorm.yusanshi.com
server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    listen 8000 ssl default_server;
    listen [::]:8000 ssl default_server;
    server_name _;
    error_page 497 https://$host:$server_port$request_uri;
    return 301 https://dorm.yusanshi.com;
}

### dorm.yusanshi.com
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    listen 8000 ssl;
    listen [::]:8000 ssl;
    server_name dorm.yusanshi.com;
    error_page 497 https://$host:$server_port$request_uri;
    root /var/www/dorm.yusanshi.com;
    location / {
        try_files $uri $uri/ =404;
    }
}

### code.dorm.yusanshi.com
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    listen 8000 ssl;
    listen [::]:8000 ssl;
    server_name code.dorm.yusanshi.com;
    error_page 497 https://$host:$server_port$request_uri;
    location / {
        proxy_pass http://localhost:8080/;
        proxy_set_header Host $host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection upgrade;
        proxy_set_header Accept-Encoding gzip;
    }
}
```

## 已知问题

- Virtual Machines 和 LAN 下的设备都能正常获得 IPv4 和 IPv6 地址（指公网/教育网地址，下同），但 Proxmox Host 无法获得 IPv4 地址，只能获得 IPv6 地址；
- 设备重启后 Wireless AP 下的设备不能访问 Proxmox Host 和 OpenWrt 外的其他 Proxmox VMs 的 IPv6 地址，需要先在 OpenWrt 中手动 ping 一下它们的 IPv6 地址：`ping -c 4 host.dorm.yusanshi.com`（Proxmox Host），`ping -c 4 dorm.yusanshi.com`（Proxmox Ubuntu VM）。
