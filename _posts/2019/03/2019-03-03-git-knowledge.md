---
layout: post
title: 代码管理的git--应对百变的需求
category: tools
tags: [tools]
---



### 分支

git 可以使用多个分支同时进行开发。一般情况，在开发团队中会有开发版本分支(dev)，发布版本分支(release)。再日常项目中，我一般还会添加一个预发布分支(pre-release)。作为预发布代码的测试。

开发分支用于做日常开发。发布分支用于发布线上代码，预发布分支用于预发布测试。正常开发流程，在开发分支完成测试环境测试之后，会合并到预发布分支上，在预发布环境进行测试。预发布环境测试完之后，再合并到发布分支上。

如果在新版本开发的过程中，线上代码有个bug需要修复，可以从预发布分支上检出一份代码，修复bug之后再合并到预发布分支上，在预发布环境测试。测试没问题再发布到线上分支。

整个过程流程图如下:

以下是实际开发的一个截图

分支相关命令如下:

<code style="font-family: Roboto, 'Courier New', Consolas, Inconsolata, Courier, monospace; background-image: initial; background-position: initial; background-size: initial; background-repeat: initial; background-attachment: initial; background-origin: initial; background-clip: initial; overflow-x: scroll; margin-right: 2px; margin-left: 2px; padding: 1px 2px; border-radius: 2px; display: inline;">## 创建新分支
git branch newbranchname 

## 切换到新分支
git checkout newbranchname

## 创建并切换到新分支
git branch -b newbranchname

## 将新分支推送到远程服务器(先进入到分支中)
git push --set-upstream origin newbranchname

## 删除分支（本地分支）
git branch newbranchname

## 强制删除（本地分支）
git branch -D newbranchname

## 删除远程分支(不会删除本地分支)
git push origin --delete newbranchname

## 查看本地分支
git branch 

## 查看所有分支(会把远程分支列出来)
git barnch -a

## 查看本地分支与远程分支关联关系
git branch -vv</code> 

分支解决了开发环境，测试环境，预发布环境代码问题。有了分支之后，开发人员可以在不同的情况下切换到不同分支进行开发。

### 合并指定内容

一般的版本开发都需要一个周期。但是有的时候，前期评估周期是一个月，但是在刚开发到半个月的时候就在想能否部分功能先上线。开发过程提倡的是频繁提交，因此一些功能散落在多次提交当中，并且每次提交都是相对的功能专一。这种情况下，比较适合单独合并指定commit。

git中cherry-pick就可以实现这个功能。cherry-pick会把commit的变更内容"复制"过来，同时在当前分支生成一个新的commit。commit hash与原来的不再相同。如果没有冲突，则commit msg 是一样的。如果有冲突，则需要自己手动解决冲突，编辑提交信息

<code style="font-family: Roboto, 'Courier New', Consolas, Inconsolata, Courier, monospace; background-image: initial; background-position: initial; background-size: initial; background-repeat: initial; background-attachment: initial; background-origin: initial; background-clip: initial; overflow-x: scroll; margin-right: 2px; margin-left: 2px; padding: 1px 2px; border-radius: 2px; display: inline;">## 合并指定的commitid到本分支
git cherry-pick commit-hash

## 合并连续的多个commit
git cherry-pick commit-start..commit-end</code> 

在测试分支上的version1的提交

在预发布分支上，使用cherry-pick version1

使用cherry-pick之后，预发布版本上的内容有了version1的提交。同时创建了一个新的commit。

### 找回历史的一个版本

需求变更是常事，经常做了一圈之后发现还是第一个版本好，这个时候想要找到以一个版本的内容。

恢复指定文件到指定版本。首要任务是先找到指定版本。可以使用git log查找指定版本。

<code style="font-family: Roboto, 'Courier New', Consolas, Inconsolata, Courier, monospace; background-image: initial; background-position: initial; background-size: initial; background-repeat: initial; background-attachment: initial; background-origin: initial; background-clip: initial; overflow-x: scroll; margin-right: 2px; margin-left: 2px; padding: 1px 2px; border-radius: 2px; display: inline;">## 查看提交记录
## (包含提交信息，时间，提交人)
## 只会显示当前分支记录
git log

## 查看所有分支记录
git log -g

## 按commit msg 查找
git log -g --grep="查找关键字"

## 按时间查找
git log --after="2019-02-27 00:00" --before="2019-02-27 12:00"

## 显示指定文件的历史记录
## (会显示文件变更内容)
git log -p filename

## 按文件内容变化查找
git log --all -S "文件内容关键字" filename</code> 

使用git log 查找到需要恢复的commitid。一般情况，由于版本开发过程，文件之间相互引用，不会直接替换成旧版本，而只是把一部分功能恢复成旧版本。

可以使用checkout将旧版本和当前版本合并。

<code style="font-family: Roboto, 'Courier New', Consolas, Inconsolata, Courier, monospace; background-image: initial; background-position: initial; background-size: initial; background-repeat: initial; background-attachment: initial; background-origin: initial; background-clip: initial; overflow-x: scroll; margin-right: 2px; margin-left: 2px; padding: 1px 2px; border-radius: 2px; display: inline;">## checkout 文件 
## -p参数显示commit变更内容
## 然后根据提示合并内容
git checkout commit-id filename -p</code> 

也可以使用git show将旧版本输出到一个文件，然后手动合并内容

<code style="font-family: Roboto, 'Courier New', Consolas, Inconsolata, Courier, monospace; background-image: initial; background-position: initial; background-size: initial; background-repeat: initial; background-attachment: initial; background-origin: initial; background-clip: initial; overflow-x: scroll; margin-right: 2px; margin-left: 2px; padding: 1px 2px; border-radius: 2px; display: inline;">git show commit-id:filename > old_version_savepath</code> 

以下是演示截图

对于线上版本，如果真的不幸，上线之后又bug无法修复，那只能是进行版本回滚。回滚到上一个发布版本的commit-id。

<code style="font-family: Roboto, 'Courier New', Consolas, Inconsolata, Courier, monospace; background-image: initial; background-position: initial; background-size: initial; background-repeat: initial; background-attachment: initial; background-origin: initial; background-clip: initial; overflow-x: scroll; margin-right: 2px; margin-left: 2px; padding: 1px 2px; border-radius: 2px; display: inline;">## 将版本回退到指定commit
git checkout commit-id</code> 

以上内容就是在需求变更使用频繁的命令。主要设计到分支操作，git log的操作，以及checkout 历史的操作。

开发过程中除了上述命令之外，还有一些其他操作，可以很好的帮助开发人员提高代码管理的效率。比如整理commit,使得发布版本变得干净。更换，添加远程地址，修改提交信息，放弃提交内容等。避免篇幅过长，放在下一篇文章写。

