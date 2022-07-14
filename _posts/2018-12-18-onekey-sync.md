---
title: Jekyll 快速同步到服务器
tags:
  - Jekyll
  - scripts
---

Jekyll 博客从 GitHub 上迁移到自己的服务器上了。现在在本地使用 `bundle exec jekyll serve` 生成 html 文件后，需要手动把 `_site` 文件夹中内容同步到服务器上。

> 目标：简化这一流程，尽可能方便快速地同步到服务器。

## 方案 1：服务器上配置 Jekyll（弃用）

### 流程

先在服务器上配置好 Jekyll 环境。

写完博客后，直接把 `_posts` 同步到服务器上，或者`git push`到 GitHub 上的远程仓库，利用 Webhooks 监控仓库变化。

### 为什么弃用

- 在服务器上配置 Jekyll 环境，一方面操作失误成本高，另一方面服务器性能较弱，再配置 Ruby 等等担心吃不消；
- 还不会用 Webhooks；
- 在服务器上配置 Jekyll 的话，为了更方便地测试，还是需要在本地配置 Jeykll，增加工作量，不如只在本地配置。

## 方案 2：本地配置 Jekyll，利用脚本同步到服务器

### 流程

在本地配置好 Jekyll 环境，写完博客后，在本地构建 `_site`，之后把 `_site` 文件夹同步到服务器上相应目录即可。

为了简化操作，查阅 WinSCP 官方文档之后，编写相应同步脚本：

#### sync.bat

```
"C:\Program Files (x86)\WinSCP\WinSCP.com" /ini=nul /script=sync_script.txt
```

#### sync_script.txt

```
open sftp://root@101.132.73.105/ -hostkey="MY_HOSTKEY"
synchronize remote C:\Users\Yu\Documents\GitHub\notes\_site /var/www/blog.yusanshi.com
close
exit
```

为了更方便地在本地测试，编写构建脚本：

#### build.bat

```
@echo off
start http://127.0.0.1:4000
bundle exec jekyll serve
```

### 效果

在本地编写完 `.md` 文件后，直接双击 `build.bat`，自动打开 http://127.0.0.1:4000 ，数秒后构建完成，刷新网页即可在本地测试。

![1546168918965](https://img.yusanshi.com/upload/20191117183808987716.png)

若效果满意，直接双击 `sync.bat`，输入密码后，即可自动将 `_site` 文件夹同步到此博客的目录 `/var/www/blog.yusanshi.com`。

![1546169107573](https://img.yusanshi.com/upload/20191117183808517372.png)
