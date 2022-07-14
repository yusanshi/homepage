---
title: 科学上网·备忘
---

## SS

### install

```bash
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log
```

### uninstall

```
./shadowsocks-all.sh uninstall
```
