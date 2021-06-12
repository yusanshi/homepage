---
title: 记一次博客迁移
date: 2019-11-17 00:00:00
---

除了这个技术博客，我还有一个自己的秘密博客。

![](https://img.yusanshi.com/upload/20191117112136483772.png)

自从建成了图床之后，就一直想把里面的图片全部换成在线图片，毕竟单单只有个`*md`文件比再加上和图片的相对链接要方便的多。

我把博客从 Jekyll 迁移到 Hero 了，迁移的时候最重要的事情大概就是替换图片链接了（即把旧的相对链接的图片换成`https://img.yusanshi.com/upload/*`），为此写了一个小脚本，如下。

```Python
import requests
import re
import os

def upload_and_get_url(path):
    files = {'file': open(path, 'rb')}
    r = requests.post('https://img.yusanshi.com/upload.php',
                      files=files, headers={'referer': 'https://img.yusanshi.com/'})
    if r.status_code == 200 and r.json()['status'] == 'ok':
        url = r.json()['msg']
        return url
    else:
        print(f"Wrong! Status code {r.status_code}, message {r.json()['msg']}")
        return ""


def convert_a_file(path):
    file = open(path, 'r', encoding='utf-8')
    text = file.read()
    file.close()
    pattern = '\(\.\./assets/.+\)'
    result = re.findall(pattern, text)
    for x in result:
        url = upload_and_get_url(x[len('(../'):-1])
        text = text.replace(x, f'({url})')
    file = open(path, 'w', encoding='utf-8')
    file.write(text)
    file.close()
    print(f"Finish file {path}")

for x in os.listdir('_posts'):
    if x.endswith('.md'):
        convert_a_file(os.path.join('_posts',x))
```

替换的效果如下图。

![](https://img.yusanshi.com/upload/20191117111714858190.png)
