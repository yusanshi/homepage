---
title: 试用 GitHub Actions
tags:
  - GitHub Actions
  - CI/CD
  - GitHub
---

## 背景

软件工程课我们的大作业中，前端的 CI/CD 由我负责。以前用 Travis CI 做这些，但是现在我想尝试下 GitHub Actions。

我的需求是，GitHub 上的 Vue.js 项目在编译、测试通过后，部署到我自己的服务器（<https://interview.yusanshi.com>）上。

## 经过

服务器端配置过程如下。

```bash
## 为 GitHub Actions 创建用户
sudo useradd -m github
sudo chown -R github:github <Website Dir>
sudo su github
cd ~
## 创建临时密钥对
ssh-keygen -t rsa -b 4096 -C "github@yusanshi.com"
cat .ssh/id_rsa.pub > .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
## 这里记录下私钥
cat .ssh/id_rsa
## 删除临时创建的密钥对
rm .ssh/id_rsa .ssh/id_rsa.pub
```

在 Settings - Secrets 中添加 Secrets `SSH_PRIVATE_KEY` 和 `TARGET_DIR`（刚才记录的私钥和网站部署的目录）。

![](https://img.yusanshi.com/upload/20200430211933556833.png)

编写 GitHub Actions 脚本。

```yaml
name: Node.js CI/CD

on:
  push:
    branches: [master]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js
        uses: actions/setup-node@v1
        with:
          node-version: "12.x"
      - run: npm install
      - run: npm run build
      # - run: npm run test:unit
      # - run: npm run test:e2e
      - name: Deploy to server
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          TARGET_DIR: ${{ secrets.TARGET_DIR }}
        run: |
          SSH_PATH="$HOME/.ssh"
          mkdir "$SSH_PATH"
          echo "$SSH_PRIVATE_KEY" > "$SSH_PATH/id_rsa"
          chmod 600 "$SSH_PATH/id_rsa"
          ssh-keyscan yusanshi.com >> "$SSH_PATH/known_hosts"
          rsync -az --delete -e "ssh -i $SSH_PATH/id_rsa" ./dist/ "github@yusanshi.com:$TARGET_DIR"
```
