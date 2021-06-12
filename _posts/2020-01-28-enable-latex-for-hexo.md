---
title: 为 Hexo 添加对 LaTeX 的支持
tags:
  - LaTeX
  - Hexo
---

# 背景

前段时间初学 LaTeX 用它[做了一道题]({% post_url 2020-01-28-random-walk-problem %})，想把它放在自己的博客上，但是我使用的主题 [cactus](https://github.com/probberechts/hexo-theme-cactus) 没有配置 LaTeX，需要我手动安装插件。

在网上搜，发现 [hexo-math](https://github.com/hexojs/hexo-math) 在我的主题上不管用，又找到了看起来能用的 [hexo-renderer-mathjax](https://github.com/phoenixcw/hexo-renderer-mathjax)，但是用后者时也遇到了些小问题。

这里把过程给记录下来。

# 记录

看到的很多页面都说要先卸载默认的 Markdown 渲染工具 `hexo-renderer-marked`，但是我实测发现就用它看起来没啥问题，所以就没换。

接着安装 `hexo-renderer-mathjax`：`npm install hexo-renderer-mathjax --save`。然后就遇到了两个小问题。

1. 转义问题

   大概是渲染工具的行为，数学式子里的 `\` 被识别成转移符号，于是乎，`$ \{X_n\} $`（$ \\{X_n\\} $）变成了 `$ {X_n} $`（$ {X_n} $），同时 `\\` 也变成了 `\`。怎么解决这个问题呢？最终我想到一个办法：写完 Markdown 后，把所有的 `\` 替换成 `\\`（当然，这样有个问题，会把 Latex 公式外的普通文本里的 `\` 也给替换了，解决这个问题也不难：识别是否在公式内即可，啥时候有空我想写一个在线转换的小网页）。

2. 需要修改 `hexo-renderer-mathjax` 包

   问题表现为在我本地用 `hexo s` 测试发现正常，但是传到网站上发现公式没法正常显示，仔细观察发现浏览器地址栏有个小叉叉，再打开控制台才意识到这个包给网页插入的 Javascript 标签是 `<script src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>`，而由于我的网站是 Https 站点，由于安全原因默认不允许从 Http 地址获取 JS 脚本。在网上找到的解决办法是找到 `node_modules` 中这个包所在的文件夹，直接找到这段 JS 代码所在的部分做修改。——`node_modules` 算是中间文件，修改中间文件感觉有点脏脏的。我的博客是使用 Travis CI 自动部署的，如果还要让我在 `.travis.yml` 再对它修改，那岂不是更脏。

   事实上，在这个包的 GitHub 仓库中，这个问题已经被解决，同时又查到 `npm install git+https://.../source.git` 可以直接从 Git 源来安装包，于是：`npm install git+https://github.com/phoenixcw/hexo-renderer-mathjax --save`，但是这么做之后又碰到一个特别 tricky 的问题 `ReferenceError: hexo is not defined`，查了查感觉是 Hexo 版本太新的问题。但是为什么 npmjs.com 的就可以呢？！对比 npm.js 上的和 GitHub 上的两个版本，发现些不同后，我把 `hexo-renderer-mathjax` 给 Fork 了一份，尝试做了一些修改，也是白给。

   最后，我只能换个思路：修改完 npm.js 上的 JS 地址后传到 GitHub 上。说干就干，`wget $(npm view hexo-renderer-mathjax dist.tarball)` 获取 npmjs.com 上的包，找到 JS 地址修改完后，传到 `old` 分支上（https://github.com/yusanshi/hexo-renderer-mathjax/tree/old ）。使用 Git 作为包的安装源时，如果要指定其它分支，要在链接后加上 `#branch_name`，即 `npm install git+https://github.com/yusanshi/hexo-renderer-mathjax.git#old --save`。最终，`package.json` 的 `dependencies` 多了这一行：`"hexo-renderer-mathjax": "git+https://github.com/yusanshi/hexo-renderer-mathjax.git#old",`。

这两个问题解决之后，就可以愉快地写 Latex 了。行内公式 `$ f_{ij} = \sum_{n=1}^{\infty} f_{ij}^{(n)} $` 显示效果：$ f*{ij} = \sum*{n=1}^{\infty} f*{ij}^{(n)} $，块公式 `$$ c(x)=\sum*{n=0}^{\infty }C*{n}x^{n} ={\frac {1-{\sqrt {1-4x}}}{2x}} $$` 显示效果：$$ c(x)=\sum*{n=0}^{\infty }C\_{n}x^{n} ={\frac {1-{\sqrt {1-4x}}}{2x}} $$

当然，别忘了转义问题（在线转换的小网页明天就写）。

> 2020.1.29 补充：写好了，https://yusanshi.com/escape.html 。
