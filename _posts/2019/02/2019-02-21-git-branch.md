---
layout: post
title: 如何更优雅地切换 Git 分支
category: tools
tags: [tools]
---

在日常开发中，我们经常需要在不同的 Git 分支之间来回切换，特别是业务需求比较多的开发人员。在分支较多的情况下，分支名的 tab 自动补全会比较糟糕，切换时我们不免需要复制或手打分支名，那么有没有更优雅的方式了呢？

<a id="more"></a>

为了提高切换 Git 分支的效率，我用 Golang 写了 `git-checkout-branch` 这个小工具，可以交互式的切换分支，并自带搜索功能，帮助你更优雅的进行分支切换。

## 概览

Github 地址：[https://github.com/royeo/git-checkout-branch](https://github.com/royeo/git-checkout-branch) ，欢迎 star。

![](https://raw.githubusercontent.com/royeo/static/master/gif/git-checkout-branch.gif)

说明：

*   使用箭头键 `↓` `↑` `→` `←` 进行移动，也支持 `j` 和 `k` 的上下移动
*   使用 `/` 切换搜索
*   按 `ctrl + c` 退出

## 安装

可以直接下载安装：

| curl -sSL https://github.com/royeo/git-checkout-branch/releases/download/v0.2.0/git-checkout-branch-`uname -s`-`uname -m` -o /usr/local/bin/git-checkout-branch && chmod +x /usr/local/bin/git-checkout-branch |

也可以使用 `go get` 安装，确保 `$GOPATH/bin` 路径在 `PATH` 中。

| go get -u github.com/royeo/git-checkout-branch |

建议为 `checkout-branch` 设置别名，例如 `cb`，这样就可以直接使用 `git cb` 来进行分支切换。

| git config --global alias.cb checkout-branch |

## 帮助

使用 `git checkout-branch help` 获取帮助信息。

| Checkout git branches more efficiently.

Usage:
 git checkout-branch [flags]

Flags:
 -a, --all          List both remote-tracking branches and local branches
 -r, --remotes      List the remote-tracking branches
 -n, --number       Set the number of branches displayed in the list (default 10)
 --hide-help    Hide the help information |

 [# Golang](../tags/Golang/) [# Git](../tags/Git/) [# Tool](../tags/Tool/)

 [Golang 标准库 log 包](../2019/01/18/golang-standard-library-log/ "Golang 标准库 log 包")

