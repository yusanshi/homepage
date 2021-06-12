---
title: Git Submodule 管理
tags:
  - Git
---

最近的一个小 [repository](https://github.com/yusanshi/ros_package) 用到了多个 submodule，而我又对它们做了修改，如何保存我对它们的修改呢？

搜索之后发现一个常见的思路是自己把那个仓库 Fork 下，这样你就对那个仓库拥有了所有权，可以直接对 Fork 的仓库做修改。但是这样有个不足在于，如果你项目下有多个仓库，你就得把这几个仓库都 Fork 下，工作量大，更重要的是，对于不在 GitHub 平台上的仓库，你还得在它对应的平台 Fork（比如我就用到了 Gitlab 上 Autoware 的数个仓库）。

又搜了搜发现可以对 submodule 打 patch，想起来之前 OS 课的组队大作业（CS140e），在一个老哥的博客里，由于课程相关代码有些陈旧，在最新的 Rust 上不能跑，于是他就修改了代码并发布了一个 patch。

顺着这个思路，在 Stackoverflow 上找到了这么一个[回答](https://stackoverflow.com/a/15438863)，搬运如下。

> If you haven't yet commited the changes, then:
>
> ```
> git diff > mypatch.patch
> ```
>
> But sometimes it happens that part of the stuff you're doing are new files that are untracked and won't be in your `git diff` output. So, one way to do a patch is to stage everything for a new commit (`git add` each file, or just `git add .`) but don't do the commit, and then:
>
> ```
> git diff --cached > mypatch.patch
> ```
>
> Add the 'binary' option if you want to add binary files to the patch (e.g. mp3 files):
>
> ```
> git diff --cached --binary > mypatch.patch
> ```
>
> You can later apply the patch:
>
> ```
> git apply mypatch.patch
> ```
>
> Note: You can also use `--staged` as a synonym of `--cached`.

总结下，可以使用 `git diff > target.patch` 生成 patch，但是这种方式不能记录对新文件的修改，于是我们可以 `git add . && git diff --cached > target.patch`，如果要添加二进制文件（比如我可能修改了 submodule 里的一张图），使用 `git add . && git diff --cached --binary > target.patch`。

出于方便，避免记录每个 patch 文件的位置，我把 patch 的位置选择了和 submodule 同级的目录，比如 `/path/to/foo/bar` 的 patch 文件将会是 `/path/to/foo/bar.patch`，这样的话，不管是打 patch，还是应用 patch，patch 文件的路径都直接 `../filename.patch` 即可。

一个一个对 submodule 打 patch、应用 patch 不方便，因此写了一个小脚步，作为初学 shell 脚本的练习。我把它放在了 https://github.com/yusanshi/utils ，并把它以 submodule 的方式添加到需要使用这个脚本的仓库里，打 patch 和应用 patch 的脚本如下。

**generate_patches.sh**

<script src="https://emgithub.com/embed.js?target=https://github.com/yusanshi/utils/blob/master/generate_patches.sh&style=tomorrow"></script>

**apply_patches.sh**

<script src="https://emgithub.com/embed.js?target=https://github.com/yusanshi/utils/blob/master/apply_patches.sh&style=tomorrow"></script>
