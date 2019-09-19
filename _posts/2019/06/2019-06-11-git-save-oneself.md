---
layout: post
title: Git 自救指南 - 知乎
category: tools
tags: [tools]
---






Git 虽然因其分布式管理方式，不完全依赖网络，良好的分支策略，容易部署等优点，已经成为最受欢迎的源代码管理方式。但是一分耕耘一分收获，如果想更好地掌握 git，需要付出大量的学习成本。即使在各种 GUI 的加持下，也不得不说 git 真的很难，在 V2EX 上也常有如何正确使用 git 的讨论，同时在 Stackoverflow 上超过 10w+ 的 git 相关问题也证明了 git 的复杂性。再加上 git 的官方文档也一直存在着 “先有鸡还是先有蛋” 的问题，虽然文档非常全面，但如果你不知道你遇到的问题叫什么，那么根本就无从查起。

<noscript>![](https://pic3.zhimg.com/v2-18d50c2c9923668f9daf1e576aca133a_b.gif)</noscript>

![](data:;base64,<svg xmlns='https://www.w3.org/2000/svg' width='800' height='167'></svg>)

作为国内领先的研发管理解决方案供应商，CODING 一直致力于在国内普及 git 的使用，为软件研发提供更高效率。本文节选自 Katie Sylor-Miller 在日常工作中所遇到过的让他很头疼的 git 相关问题，并整理了相应的应对措施，在这里分享给正在学习如何使用 git 的同学们。当然这些应对措施并不是唯一的，可能你会有其他更好的应对方法，这也恰恰是 git 这套版本控制系统强大的地方。

> 原文标题：《Oh shit，git！》
> 原文地址：[https://ohshitgit.com/](https://link.zhihu.com/?target=https%3A//ohshitgit.com/)

*   **我刚刚好像搞错了一个很重要的东西
    但是 git 有个神奇的时间机器能帮我复原！**

<noscript>![](https://pic4.zhimg.com/v2-1198e5a04d49934552c6e72131f0fc87_b.jpg)</noscript>

![](data:;base64,<svg xmlns='https://www.w3.org/2000/svg' width='1504' height='594'></svg>)

reflog 是一个非常实用的命令，你可以使用这个命令去找回无意间删除的代码，或者去掉一些刚刚添加的却把仓库里的代码弄坏的内容。同时也可以拯救一下失败的 merge，或者仅仅是为了回退到之前的版本。

*   **我 commit 完才想起来还有一处小地方要修改！**

<noscript>![](https://pic2.zhimg.com/v2-cbc7e0e449d1d1be3b198f69da6eed1d_b.jpg)</noscript>

![](data:;base64,<svg xmlns='https://www.w3.org/2000/svg' width='1504' height='522'></svg>)

当我 commit 完然后跑测试的时候，经常突然发现忘了在等于号前面加空格。虽然可以把修改过的代码再重新 commit 一下，然后 rebase -i 将两次揉在一起，不过上面的方法会比较快。

*   **我要改一下上一个 commit message！**

<noscript>![](https://pic4.zhimg.com/v2-ef535cde3fac85d278c2ceadf2aae49b_b.jpg)</noscript>

![](data:;base64,<svg xmlns='https://www.w3.org/2000/svg' width='1504' height='414'></svg>)

当你们组对 commit message 有格式要求时，或者当你忘了中英文间要加空格，这个命令能救你狗命。

*   **我不小心把本应在新分支上的内容 commit 到 master 了！**

<noscript>![](https://pic1.zhimg.com/v2-b2143b4fbfb9422b577674a15fa26ee8_b.jpg)</noscript>

![](data:;base64,<svg xmlns='https://www.w3.org/2000/svg' width='1504' height='558'></svg>)

注意：这个指令必须在错误的 commit 后直接执行，如果你已经试了其他的方式，你可能就需要用 git reset HEAD@{number} 来代替 HEAD~ 了。

*   **Oh shit，我不小心 commit 到错误的分支上了！**

<noscript>![](https://pic4.zhimg.com/v2-bd1a9c0db04e83b048a1038bbf7ae613_b.jpg)</noscript>

![](data:;base64,<svg xmlns='https://www.w3.org/2000/svg' width='1504' height='666'></svg>)

也有很多人推荐了 cherry-pick 的解决方案，所以选哪个就看你心情了。

<noscript>![](https://pic2.zhimg.com/v2-d89dd5ca0be844144e2b608bdfbc6ea9_b.jpg)</noscript>

![](data:;base64,<svg xmlns='https://www.w3.org/2000/svg' width='1504' height='558'></svg>)

*   **我执行了 diff 但是啥也没出现**

<noscript>![](https://pic1.zhimg.com/v2-1c7beec08229226ff00b51c319f14b6c_b.jpg)</noscript>

![](data:;base64,<svg xmlns='https://www.w3.org/2000/svg' width='1504' height='378'></svg>)

Git 不会给通过 add 加入到 staging 区域里面的文件做 diff ，除非你加了 --staged 的标签，别怀疑了这是一个 feature 不是一个 bug，当然对于第一次碰到这个问题的人来说还是有些不好理解的。

<noscript>![](https://pic2.zhimg.com/v2-d89dd5ca0be844144e2b608bdfbc6ea9_b.jpg)</noscript>

![](data:;base64,<svg xmlns='https://www.w3.org/2000/svg' width='1504' height='558'></svg>)

*   **Git 从入门到放弃**

<noscript>![](https://pic3.zhimg.com/v2-7c870e9f4e5da404365e3226a2b1a52e_b.jpg)</noscript>

![](data:;base64,<svg xmlns='https://www.w3.org/2000/svg' width='1504' height='486'></svg>)

为了维护最后的尊严 XD

不知道你在使用 git 中有没有遇到过各种令人掀桌的问题呢？
或者作为 git 资深用户有什么可以分享的小技巧呢？
欢迎大家在留言区跟我们互动～

最后，点击**[使用 CODING](https://link.zhihu.com/?target=https%3A//coding.net/)**，体验敏捷研发！

 

