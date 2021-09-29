---
title: Handwriting Go Away AGAIN
tags:
  - LaTeX
  - Handwriting
  - Scripts
---

# 背景
疫情在家的时候为了做信息安全导论的作业，写过一个叫 [Handwriting Go Away](https://github.com/yusanshi/handwriting-go-away) 的小项目，但是那个主要是面向纯文本的；最近机器学习课又有了奇怪规定：只能手写，但机器学习课里面显然会有很多公式，我打算用 LaTeX 来写，因此还需要一个面向 LaTeX 的仿手写文档生成器。

于是有了这个项目：[Handwriting Go Away AGAIN](https://github.com/yusanshi/handwriting-go-away-again)。


# 已实现效果
- 换字体为手写体（中文，英文和数学符号）；
- 换背景为纸张照片；
- 减小页面边距；
- 增大字体；
- 移除页码；
- 移除段落缩进；
- 深灰色文字；
- 随机微旋页面；
- 移除公式编号；
- 压缩、加噪、降 DPI。

# 效果对比

原始文档：

<embed type="application/pdf" src="https://storage.yusanshi.com/handwritinggoaway-again/hw1.pdf" width="800" height="1200">

<br>

转换后文档：

<embed type="application/pdf" src="https://storage.yusanshi.com/handwritinggoaway-again/output.pdf" width="800" height="1200">

<br>

# 不足

目前同样的两个字符显示是一模一样的，我希望能给字体创造出多个“相似”的字体，生成文档的时候每个字随机在多个相似字体中选择一个。为了生成多个“相似”字体，目前有两种思路：
- 字体转为 SVG 文件，做随机变换，然后再合成字体文件；
- 使用深度学习方法（具体来说可能特指 GAN）来生成新字体。

目前两种思路都没有实现，我对第二种方法比较感兴趣。
