---
title: 利用 Travis CI 自动部署 Hexo 博客
tags:
  - Travis
  - CI/CD
---

## 背景

嫌 GitHub Pages 国内访问太慢，而且我又有已备案的域名和服务器，因此没有使用 `hexo deploy` 直接部署到 GitHub 的方式，而是使用 Nginx 提供对 `hexo generate` 生成的静态文件的访问。

使用前者可以用 `hexo deploy` 一键部署，但是使用后者却往往只能手动把生成的静态文件放到 Nginx 中网页对应的目录里，很麻烦。之前一直听说过 Travis CI，但是没有用过，趁着这次机会学习了 Travis CI，最终实现了一键部署（确切地说，是 Git Push 后自动部署），记录如下。

## 基本思路

### 准备

1. 建立 GitHub 仓库，里面放入你的 Markdown 文件；

2. 在此仓库配置好 Travis CI 自动部署（授权 Travis CI 对仓库的读写权限、编写 `.travis.yml`）；
3. 为网站目录分配新用户，配置密钥使得 Travis CI 可以用 SSH 连接到网站服务器并修改这个目录（为了能够使用 `rsync`）。

### 工作流程

1. 在本地写 Markdown 文件，使用 `hexo serve` 在本地测试效果，若效果满意，Push 到 GitHub；

2. Travis CI 监测到 Push 操作后，自动触发 Build 流程，生成静态文件（`public`），使用 `rsync` 将生成的文件同步到网站目录。

其中第 2 步的行为由 `.travis.yml` 来定义。

## 开始

以下操作是在博客服务器上进行的。

### 安装 Travis CLI

```bash
sudo apt update
sudo apt install ruby-full
gem install travis
```

### 新建用户

```bash
sudo useradd -m travis
chown -R travis:travis <Blog Dir> # 替换 <Blog Dir> 为你的网站目录
su travis
cd ~
```

### 绑定 Travis CI

我通过 [GitHub Student Developer Pack](https://education.github.com/pack) 获得了 https://travis-ci.com/ 上的对私有仓库的使用权限。使用 Travis CI 前要授权它对你的仓库的相关权限，这里摘录 Travis CI 文档里的步骤。

1. Go to [Travis-ci.com](https://travis-ci.com/) and [_Sign up with GitHub_](https://travis-ci.com/signin).
2. Accept the Authorization of Travis CI. You’ll be redirected to GitHub.
3. Click the green _Activate_ button, and select the repositories you want to use with Travis CI.

我只授予了一个仓库——我的博客的访问权限。GitHub 的 Installed GitHub Apps 显示如下。

![](https://img.yusanshi.com/upload/20191125164747999498.png)

![](https://img.yusanshi.com/upload/20191125164806795074.png)

### 配置密钥对

#### 方案一：使用新建的密钥对

##### 新建临时密钥对

```bash
su travis
cd ~
ssh-keygen -t rsa -b 4096 -C "travis@yusanshi.com"
```

##### 将公钥添加到 `authorized_keys`

```bash
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
```

##### 加密私钥

```bash
travis login --com
travis encrypt-file .ssh/id_rsa -r yusanshi/blog # 替换 yusanshi/blog 为你的 [GitHub username]/[repo name]，下同
```

输出如下。

```
encrypting .ssh/id_rsa for yusanshi/blog
storing result as id_rsa.enc
storing secure env variables for decryption

Please add the following to your build script (before_install stage in your .travis.yml, for instance):

    openssl aes-256-cbc -K $encrypted_2c430460807a_key -iv $encrypted_2c430460807a_iv -in id_rsa.enc -out .ssh/id_rsa -d

Pro Tip: You can add it automatically by running with --add.

Make sure to add id_rsa.enc to the git repository.
Make sure not to add .ssh/id_rsa to the git repository.
Commit all changes to your .travis.yml.
```

按照提示修改 `.travis.yml`，并将当前目录下生成的 `id_rsa.enc` 放到仓库根目录。

##### 删除临时密钥对

```bash
rm .ssh/id_rsa .ssh/id_rsa.pub
```

最终，该方案在 `.travis.yml` 下添加了如下行。

```
before_install:
  - openssl aes-256-cbc -K $encrypted_2c430460807a_key -iv $encrypted_2c430460807a_iv -in id_rsa.enc -out .ssh/id_rsa -d
  - chmod 700 .ssh
  - chmod 600 .ssh/id_rsa
  - eval "$(ssh-agent -s)"
  - ssh-add .ssh/id_rsa
```

#### 方案二：直接使用仓库的公钥

方案一有点麻烦，我用方案一弄完之后才发现[这个](https://docs.travis-ci.com/user/encryption-keys#fetching-the-public-key-for-your-repository)，可以获得仓库的公钥，然后将其添加到 `authorized_keys` 即可。

> 后来我将自己的仓库 Make Public 之后发现这种方式 `rsycn` 还是会提醒我输入密码，再次 Make Private 才解决。暂时不知道怎么为 Public 的仓库用这种方式。

##### 获取仓库公钥

```bash
travis pubkey -r yusanshi/blog
```

输出如下。

```bash
Public key for yusanshi/blog:

ssh-rsa ......
```

##### 将公钥添加到 `authorized_keys`

```bash
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
vim .ssh/authorized_keys # 添加上面的公钥
```

使用该方案，无需方案一的添加 before_install 的步骤。

### 加密环境变量

这里我对博客目录加密。

```bash
travis encrypt TARGET_DIR="<Blog Dir>" -r yusanshi/blog # 替换 <Blog Dir> 为你的网站目录
```

按照提示修改 `.travis.yml` 即可。

### 总结

这是我在用方案二的情况下最终的 `.travis.yml` 文件内容，供参考。如果你用方案一（网上的教程大多是这个），记得添加 before_install 这一项。

```yaml
language: node_js
node_js: stable

env:
  - secure: "FBW7i4vslEd0SCpnrKvSfWgQS0MU+KSdPhUA4lDgUUFpXjemugEG6bVX9MYXi2bFfV60WEyDzWWYH6lDB44Y98YPbneeL6W0OsVR7gJ9KPklRF55/1DmrDAAY0KtmqK+aZLBxCWAx5a9w6RNgZ6xx2l+KO/NLnyGSc9/RH4kn3wglDs6I1K3EAc++0xk0thqomhoXJo4M8NeX3+BjoTeaog648pYv+W/15sWczaDBpy76sGxliGRZRCXirRFasQzl3rihK/EtkClMIC7bbvlqkDdzWh0TiI7rp103vDaVXhJPsTs4RZnQ/+I+fnrDSc3oAmVxELL3XSpsBuyiW4gW5Y0o5j8mZ0vJlMFFsodA8b74VQdU8vLdon5NHjaSTXYZlefxo3450p1V4873ZXEthVisULLmTBjf3cz79IelMnzKV5W0Auhu5YpbUN0hdmgiGGLnVAIjDi391CnZW9OLq75vo5kPNNBLfnCkhkeiLAXT/l1omMBniMwsjPFuV7dl5gSjRRtysXmYIHNxe0MTriWcOeMebSaEWEQu+YLpgICvJ7+p97W2c93a1PN59rhY4as9T9xzjGT5XphLgXrxiHxYLw2ZZx/W3NtceY8iON65UVTgwQwtypruxwn+sASNHmZlU5WoafNzdn82xrtYdhIUDj2ryd2VYMMST/NCUk="

addons:
  ssh_known_hosts: yusanshi.com

install:
  - npm install

before_script:
  # Apply my modifications to cactus theme
  - cp ./themes/cactus_modify/* ./themes/cactus/ -r

script:
  - hexo generate

after_success:
  - rsync -az ./public/* travis@yusanshi.com:$TARGET_DIR
```

## 遇到的错误

> 以下错误都是我用方案一（新建密钥对）时遇到的，因此更推荐方案二。

### No such file or directory:bss_file.c:398:fopen('.ssh/id_rsa','w')

```
140046600656536:error:02001002:system library:fopen:No such file or directory:bss_file.c:398:fopen('.ssh/id_rsa','w')
140046600656536:error:20074002:BIO routines:FILE_CTRL:system lib:bss_file.c:400:
The command "openssl aes-256-cbc -K $encrypted_2c430460807a_key -iv $encrypted_2c430460807a_iv -in id_rsa.enc -out .ssh/id_rsa -d" failed and exited with 1 during .
```

一番搜索但是没找到结果。抱着试一试的心态，我在 before_install 里加了一行 `ls .ssh`，再次 Build，发现在这里报错：找不到 `.ssh` 目录，莫非上面的问题也是由于找不到这个目录？试了下在仓库里新建了文件夹 `.ssh` ，里面放上空文件 `.gitkeep`，就可以了，看来果然如此。

### 已配置 SSH Key 但仍提示输入密码

在 before_install 末尾添加如下两行。

```
  - eval "$(ssh-agent -s)"
  - ssh-add .ssh/id_rsa
```
