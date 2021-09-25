---
title: All in One Dorm Server 配置记录
tags:
  - Router
  - Server
  - Nginx
  - Proxmox
  - OpenWrt
---

最近看服务器/软路由心潮澎湃，想买一台 HPE MicroServer Gen10 Plus，奈何手头有点紧张，一番思考之后，决定先买个更便宜的设备，等以后有了自己的房子再来上大家伙。于是就在淘宝选了个 J4125 的软路由。因为手头有两条镁光内存条（8GB DDR4 2400），就选了准系统版本并且额外买了个 mSATA 的固态硬盘（我手边有 M2 的固态奈何那个软路由只有 mSATA 接口）。



# 硬件

| 项目 | 型号                   |
| ---- | ---------------------- |
| CPU  | Intel Celeron J4125    |
| 内存 | 镁光 8GB DDR4 2400 x 2 |
| 硬盘 | 三星 850EVO 250G mSATA |
| 网卡 | Intel I210 x 4         |



**俯视图**

![](https://img.yusanshi.com/upload/20210923185410850842.jpg)



# 软件

**Host**

Proxmox VE 7.0-2

**Virtual Machines**

- OpenWrt 21.02.0 x86-64

- Ubuntu 20.04.3



<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" version="1.1" viewBox="0 0 611 482" content="&lt;mxfile host=&quot;app.diagrams.net&quot; modified=&quot;2021-09-23T11:55:56.790Z&quot; agent=&quot;5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/93.0.4577.82 Safari/537.36&quot; etag=&quot;Bl5ZNDKXuHIo08aRoOjL&quot; version=&quot;15.3.0&quot; type=&quot;google&quot;&gt;&lt;diagram id=&quot;LaNglwaAvbg8P3oUQVeL&quot; name=&quot;Page-1&quot;&gt;7Vvbdps4FP0aP7oLiZt5dOyks9q5pOPpyuSRMaphBSMX5Bj360cYiYuk2G5tQPXqi4OO0G3vzdHhiIzM2Tp/n/qb8A8coHgEjSAfmfMRhC6c0N/CsC8NlmeVhlUaBaUJ1IZF9A0xo8Gs2yhAWetGgnFMok3buMRJgpakZfPTFO/at33BcXvUjb9CkmGx9GPZ+hQFJCytE9uo7b+haBXykYHBatY+v5kZstAP8K5hMu9H5izFmJRX63yG4gI7jkvZ7uGN2mpiKUrIOQ0+xel0hxcRNmbPEI//Scj229gse3n14y1bMJss2XMEUrxNAlR0YozMu10YEbTY+Muidkcpp7aQrGNaAvRSnhSb5ytKCcobJjbJ9wivEUn39BZW6zC8mGCAy8q7Gn6b3xM2oDctZvQZ5auq6xoVesGA+Q6QbAVITkyHvfuC6TKbaDlft5hXjLODmqf0BuBs8rqSXq2Kv39tUPKUEt4ZnVvZX1k7OBHAFpjwZCagoWCiegauzsSkIyY+/7el/etKhOmdJqKy9UOEJxGBSGgMjhQAbaQsW0bKUgAFu8KJz6cNFBgeKFc3oIAKqOF3o2rF2gAFVUCZwwOlnaLk2EYPHyVsqyYYGihLTx8FgW5AyYGgHj7K0Q0oR08fZWqnKFdPHyVGnMMD1VXk/5jifI3zc0N/ih9po52RFL+gGY5xSi0JTlAxdBTHgsmPo1VCi0vKCqL2u4KNaOnHU1axjoKgGEZJb1sAV2DYFnyGNRmYYSjHNb9P/9SUgMDPwkMBMJ3xsUbQfHiYe47TkWe3hmZJDqqefkqWPHduuG5HEd3gLMkRnUQRSoJpkR+toW/B1aAO5RH5t4D0nc1Kzwzg4nqeNwt7XkjoOhqNiuIz768o1M0OJd7uTTYyvE2X6MiaWUaC+OkKkWM7CVMwClqJX5ncZr5RQR63pSj2SfTaTherGGUjPOLosF8x7VhiSCKKolw4a1XrQuoI2pN3lgNsc1L+2q5ak7zfEiip34PeKhQukKAcAh6JasDpqEbhK9o+5eB6723Pq2p4wh527KQNgUJDfvyBKmPdWcIaymHlZeAr8DyDjjvv3pjaV/Kx1mmQ+8VYFZHeuI/lvuq0k7W0crLiBm05P+hkweSNnb4vryrn3W9fdPzY96TobK1EJ24LPyw6KLq+nkVnymcYty86eK7oHL1Edy1PB8UMUN+ik8+DLhHdtQV0WheuVroQw6fqdEbjHXCxf5l8mC+Mjfdp/JJ9+Ag+//11LMviJrJEUiCsUN35Z5Q95vKUJMmpvJtIEl1EkvRpwtAkfdenX0O/oF8GvZjrVuTnOntBV2J/Rn6u//fzyzAWo10Fxr1CLJ/VPkV0E0VZRq3TxyN4n33Edhlgwo5sK76XAKqv6zpD7IyMnc5hvhBpnYz7j6nmdHhnqMn9yV4txWjO7jmau3aeUuM9yHwD6ebjDhWSEam92uMu5y9/RdLi63N1Sn79II0W62//y+ep/gcK8/5/&lt;/diagram&gt;&lt;/mxfile&gt;" style="background-color: rgb(255, 255, 255);"><defs/><g><rect x="25" y="25" width="560" height="340" fill="#ffffff" stroke="#000000" pointer-events="all"/><rect x="115" y="45" width="200" height="100" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 198px; height: 1px; padding-top: 95px; margin-left: 116px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;"><font style="font-size: 16px">OpenWrt</font></div></div></div></foreignObject><text x="215" y="99" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">OpenWrt</text></switch></g><rect x="355" y="45" width="190" height="100" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 188px; height: 1px; padding-top: 95px; margin-left: 356px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;"><font style="font-size: 16px">Ubuntu</font></div></div></div></foreignObject><text x="450" y="99" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">Ubuntu</text></switch></g><rect x="75" y="305" width="40" height="20" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 315px; margin-left: 76px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">eth0</div></div></div></foreignObject><text x="95" y="319" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">eth0</text></switch></g><rect x="135" y="305" width="40" height="20" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 315px; margin-left: 136px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">eth1</div></div></div></foreignObject><text x="155" y="319" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">eth1</text></switch></g><rect x="185" y="305" width="40" height="20" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 315px; margin-left: 186px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">eth2</div></div></div></foreignObject><text x="205" y="319" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">eth2</text></switch></g><rect x="235" y="305" width="40" height="20" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 315px; margin-left: 236px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">eth3</div></div></div></foreignObject><text x="255" y="319" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">eth3</text></switch></g><rect x="115" y="165" width="40" height="20" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 175px; margin-left: 116px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">eth0</div></div></div></foreignObject><text x="135" y="179" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">eth0</text></switch></g><rect x="175" y="165" width="40" height="20" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 175px; margin-left: 176px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">eth1</div></div></div></foreignObject><text x="195" y="179" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">eth1</text></switch></g><rect x="225" y="165" width="40" height="20" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 175px; margin-left: 226px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">eth2</div></div></div></foreignObject><text x="245" y="179" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">eth2</text></switch></g><rect x="275" y="165" width="40" height="20" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 175px; margin-left: 276px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">eth3</div></div></div></foreignObject><text x="295" y="179" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">eth3</text></switch></g><rect x="355" y="165" width="40" height="20" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 175px; margin-left: 356px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">eth0</div></div></div></foreignObject><text x="375" y="179" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">eth0</text></switch></g><rect x="525" y="335" width="40" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 345px; margin-left: 526px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;"><font style="font-size: 16px">Proxmox</font></div></div></div></foreignObject><text x="545" y="349" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">Proxmox</text></switch></g><rect x="225" y="195" width="40" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 205px; margin-left: 226px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 217, 102); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">LAN</div></div></div></foreignObject><text x="245" y="209" fill="#FFD966" font-family="Helvetica" font-size="12px" text-anchor="middle">LAN</text></switch></g><rect x="115" y="195" width="40" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 205px; margin-left: 116px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(151, 208, 119); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">WAN</div></div></div></foreignObject><text x="135" y="209" fill="#97D077" font-family="Helvetica" font-size="12px" text-anchor="middle">WAN</text></switch></g><path d="M 95 305 L 135 185" fill="none" stroke="#000000" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"/><rect x="165" y="155" width="160" height="40" rx="6" ry="6" fill="none" stroke="#ffe599" stroke-width="2" pointer-events="all"/><rect x="105" y="155" width="60" height="40" rx="6" ry="6" fill="none" stroke="#b9e0a5" stroke-width="2" pointer-events="all"/><path d="M 155 305 L 195 185" fill="none" stroke="#000000" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"/><path d="M 205 305 L 245 185" fill="none" stroke="#000000" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"/><path d="M 255 305 L 295 185" fill="none" stroke="#000000" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"/><path d="M 105 305 L 375 185" fill="none" stroke="#000000" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"/><rect x="185" y="335" width="40" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 345px; margin-left: 186px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 217, 102); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">LAN</div></div></div></foreignObject><text x="205" y="349" fill="#FFD966" font-family="Helvetica" font-size="12px" text-anchor="middle">LAN</text></switch></g><rect x="75" y="335" width="40" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 345px; margin-left: 76px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(151, 208, 119); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">WAN</div></div></div></foreignObject><text x="95" y="349" fill="#97D077" font-family="Helvetica" font-size="12px" text-anchor="middle">WAN</text></switch></g><rect x="125" y="295" width="160" height="40" rx="6" ry="6" fill="none" stroke="#ffe599" stroke-width="2" pointer-events="all"/><rect x="65" y="295" width="60" height="40" rx="6" ry="6" fill="none" stroke="#b9e0a5" stroke-width="2" pointer-events="all"/><rect x="105" y="405" width="100" height="40" fill="#ffffff" stroke="#000000" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 98px; height: 1px; padding-top: 425px; margin-left: 106px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(0, 0, 0); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">Wireless AP</div></div></div></foreignObject><text x="155" y="429" fill="#000000" font-family="Helvetica" font-size="12px" text-anchor="middle">Wireless AP</text></switch></g><path d="M 155 405 L 155 325" fill="none" stroke="#000000" stroke-miterlimit="10" stroke-dasharray="3 3" pointer-events="stroke"/><rect x="95" y="395" width="120" height="60" rx="9" ry="9" fill="none" stroke="#ffe599" stroke-width="2" pointer-events="all"/><rect x="215" y="415" width="40" height="20" fill="none" stroke="none" pointer-events="all"/><g transform="translate(-0.5 -0.5)"><switch><foreignObject pointer-events="none" width="100%" height="100%" requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility" style="overflow: visible; text-align: left;"><div xmlns="http://www.w3.org/1999/xhtml" style="display: flex; align-items: unsafe center; justify-content: unsafe center; width: 38px; height: 1px; padding-top: 425px; margin-left: 216px;"><div style="box-sizing: border-box; font-size: 0px; text-align: center;"><div style="display: inline-block; font-size: 12px; font-family: Helvetica; color: rgb(255, 217, 102); line-height: 1.2; pointer-events: all; white-space: normal; overflow-wrap: normal;">LAN</div></div></div></foreignObject><text x="235" y="429" fill="#FFD966" font-family="Helvetica" font-size="12px" text-anchor="middle">LAN</text></switch></g></g><switch><g requiredFeatures="http://www.w3.org/TR/SVG11/feature#Extensibility"/><a transform="translate(0,-5)" xlink:href="https://www.diagrams.net/doc/faq/svg-export-text-problems" target="_blank"><text text-anchor="middle" font-size="10px" x="50%" y="100%">Viewer does not support full SVG 1.1</text></a></switch></svg>



# 配置文件

## Host

### `/etc/network/interfaces`

这个配置文件里的 `eno1` 的别名是 `enp3s0`，但是我不知道怎么把它还原成 `enp3s0`（或者说怎么优雅地做这件事，我找到的方案很复杂）。

```
# /etc/network/interfaces
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

### `host.dorm.yusanshi.com.conf`

```
ssl_certificate /root/cert/fullchain.pem;
ssl_certificate_key /root/cert/privkey.pem;


## http -> https
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

## host.dorm.yusanshi.com
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



## Virtual Machines

### OpenWrt

#### `/etc/config/network`

```
# /etc/config/network
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

#### `/etc/config/dhcp`

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
# /etc/config/dhcp
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

### Ubuntu

#### `/etc/netplan/00-installer-config.yaml`

```
# /etc/netplan/00-installer-config.yaml
network:
  ethernets:
    ens18:
      dhcp4: true
      dhcp6: true
  version: 2
```

#### `dorm.yusanshi.com.conf`

```
ssl_certificate /home/yu/cert/fullchain.pem;
ssl_certificate_key /home/yu/cert/privkey.pem;

## http -> https
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _ ;
    return 301 https://$host$request_uri;
}

## *.dorm.yusanshi.com
server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    listen 8000 ssl default_server;
    listen [::]:8000 ssl default_server;
    server_name _;
    error_page 497 https://$host:$server_port$request_uri;
    return 301 https://dorm.yusanshi.com;
}

## dorm.yusanshi.com
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

## code.dorm.yusanshi.com
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

# 已知问题

- Virtual Machines 和 LAN 下的设备都能正常获得 IPv4 和 IPv6 地址（指公网/教育网地址，下同），但 Proxmox Host 无法获得 IPv4 地址，只能获得 IPv6 地址。
