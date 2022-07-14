---
title: LaTeX 生成 SVG 文件
tags:
  - LaTeX
---

## 背景

用 Markdown 写作业的时候，需要用 LaTeX 画一些图，然后在 Markdown 中引用 LaTex 生成的图。

LaTeX 原生生成的文件是 PDF，直接用 HTML 代码也可以嵌入 PDF 文件（使用 `object`、`embed` 等标签，见 [Stack Overflow](https://stackoverflow.com/questions/291813/recommended-way-to-embed-pdf-in-html)），但是我用的 Markdown 客户端 Typora 不支持这个语法，因此需要想办法生成图片，在 Markdown 中引入图片。

## 解决

### v1.0

Google 后发现我用的 [`standalone`](http://www.ctan.org/pkg/standalone) 类可以直接导出图片，做法是 LaTeX 文件中加上 `convert` 选项，编译时加上 `-shell-escape`，安装 Image Magick，详见 <https://tex.stackexchange.com/questions/51757/how-can-i-use-tikz-to-make-standalone-svg-graphics>。

但我发现这种方法生成的图片文件，哪怕指定了 SVG 格式，它还是会把 PDF 文件栅格化，抛弃了 LaTeX 生成的 PDF 文件的矢量性会有两个缺点：一是图片尺寸会过大，二是放大会有锯齿，看着不舒服，见下图。

![](https://img.yusanshi.com/upload/20200330171007609532.png)

为了尽可能减小锯齿，我把 density 设置的很大：

```latex
\documentclass[tikz,convert={density=600}]{standalone}
```

### v2.0

上面的解决办法不让人舒服，需要找到避免 rasterize（栅格化）的转换方法。搜索”PDF to SVG“，找到的在线转换网站里面，有几个是把 PDF 给 rasterize，但也有很多是避免 rasterize 直接生成矢量化的图形的。

这个回答 <https://tex.stackexchange.com/a/51766/211175> 指出了 convert 是可以不用 Image Magick、而用自定义命令的，那么只要找到一个可用的命令来免栅格化转换 PDF 到 SVG，再把这个命令加入到 convert 里面即可。

#### 免栅格化转换

一番搜索，发现了三个好用的东西，都可以实现我的需求（免栅格化转换 PDF 到 SVG）。

1. <https://github.com/dawbarton/pdf2svg>：Windows 下直接下载二进制文件即可，Linux 下更简单，使用包管理器安装 `pdf2svg` 即可，使用方法 `pdf2svg <input.pdf> <output.svg>`；

2. <https://www.mupdf.com/docs/manual-mutool-convert.html>：直接下载二进制文件即可，使用方法 `mutool convert -O text=path -o output.svg input.pdf`；

3. <https://github.com/pramodhkp/pdf2svg>：运行环境是 Node.js，使用方法 `node pdf2svg.js input.svg`。

一番比较，我先把 2 放弃，因为它的设定有个智障的地方：即使对于单页 PDF 文件，它输出的文件名也会有个页码“1”，`mutool convert -O text=path -o output.svg input.pdf` 这条命令实际上会生成 `output1.svg`，当然你也可以使用 `%d` 指定页码的显示方式，但是无论如何，页码都不能省去。

然后是 1 和 3，1 更简单，但是我发现 3 有个奇妙的特性：它生成的 SVG 的文字是可以选择的，而 1 不能，我又挺喜欢 JavaScript、Node.js 这一套，再加上 JavaScript 天然的跨平台性，最终我选择了更麻烦的 3。

我把原项目 Fork 后，精简了一些东西，又做了些修改，项目地址在 <https://github.com/yusanshi/pdf2svg>。运行 `npm link` 之后，就可以使用 `pdf2svg pdfPath [outputPath]` 来转换文件了（若省略 `outputPath`，则默认值是 `./`）。

#### convert 使用自定义命令

参考 <https://tex.stackexchange.com/a/51766/211175>，使用 `convert={command=...}` 即可。

```bash
\documentclass[tikz,convert={outext=.svg,command=\unexpanded{pdf2svg \infile}}]{standalone}
```

> 这里的 `outext` 不是必须的，它只是告诉 standalone：“我将要生成的文件是 `filename.svg`，我转换完成后你记得检查一下看看是不是有这个文件”，如果不加的话，`pdf2svg` 仍将正常生成 SVG 文件，只是 standalone 不知道怎么确认是否生成成功，于是报 warning：`Class standalone Warning: Conversion unsuccessful...`。

终于，现在只要编译 LaTeX 就可以生成同名的免栅格化的 SVG 文件了。
