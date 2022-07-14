---
title: 一行命令解压缩
---

## 需求

解压 `曲谱` 文件夹内所有 `rar` 文件到对应目录内。

![1563705626784](https://img.yusanshi.com/upload/20191117183809786402.png)

## 运行

```bash
find . -name '*.rar' -execdir unrar e -o+ {} \;
```

![1563705575497](https://img.yusanshi.com/upload/20191117183809828115.png)
