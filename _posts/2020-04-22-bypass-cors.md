---
title: 绕过 CORS
date: 2020-04-22 16:58:00
---

# 背景

我们的网课用的平台是 classin（<https://www.eeo.cn/>），那里有课程的回放，我的一个室友想让我帮他下载回放。

![](https://img.yusanshi.com/upload/20200426234147972095.png)

其实 classin 平台的视频播放非常友好，可以直接在代码里看到 mp4 直链。

![](https://img.yusanshi.com/upload/20200426234712592087.png)

或者是用一些插件，也很容易获得下载链接（比如下面我用的 IDM）。

![](https://img.yusanshi.com/upload/20200426234440812815.png)

但是我那位室友不是学计算机的，让他用控制台、或者下载 IDM 这类软件，对他不友好，因此我就想写个网页一键获取视频真实下载地址。

使用调试工具，很容易可以摸清视频真实下载地址的获取方式：以 `POST` 方式请求 `https://www.eeo.cn/saasajax/webcast.ajax.php?action=getLessonLiveInfo`，参数 `{lessonKey: XXXX}`，这个接口没有权限控制。

网页本身比较简单，用 Bootstrap 写得也挺快，但是在测试的时候，我才注意到 CORS（Cross-origin resource sharing，跨域资源共享）的问题。

于是我就开始寻求解决方案...

# 经过

关于 CORS，可以参考 MDN 这里的文章 <https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS>。

在谷歌搜索“bypass CORS”之后，找到了一个觉得很好的文章：<https://gist.github.com/jesperorb/6ca596217c8dfba237744966c2b5ab1e>，我决定使用里面的 [Proxy 方式](https://gist.github.com/jesperorb/6ca596217c8dfba237744966c2b5ab1e#proxy)。

先用 CORS 代理网站 <https://cors-anywhere.herokuapp.com/> 做个测试，测试结果完全 OK：

![](https://img.yusanshi.com/upload/20200422172453210805.png)

于是，我用 <https://github.com/Rob--W/cors-anywhere/> 搭建了一个简易的 CORS 代理网站 <https://cors.yusanshi.com/>，不想把它做成一个开放的代理，我在 `originWhitelist` 里加上了 `https://yusanshi.com`。

![](https://img.yusanshi.com/upload/20200427003151284156.png)

最终的下载网站在 <https://yusanshi.com/eeo-downloader.html>。

![](https://img.yusanshi.com/upload/20200427003503543434.png)

![](https://img.yusanshi.com/upload/20200427003639442894.png)

![](https://img.yusanshi.com/upload/20200422225354554328.gif)
