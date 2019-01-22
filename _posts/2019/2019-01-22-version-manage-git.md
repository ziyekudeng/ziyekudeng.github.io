---
layout: post
title: 20 分钟教你搞懂 Git！
no-post-nav: true
category: life
tags: [life]
---
尽管每天你都会用到Git，常用的命令可能不到5个，但你可能现在还搞不懂它的工作原理。为什么Git可以管理版本？基本命令git add和git commit到底在干什么？
在这篇文章中，我将用一个例子来解释Git的运行过程，帮助你理解Git的工作原理。

![](https://ziyekudeng.github.io/assets/images/2019/0122/versionManageGit/1.webp)

## **1. 初始化**
让我们创建一个项目的目录，然后进入该目录。

    $ mkdir git-demo-project
    
    $ cd git-demo-project
    

如果想管理项目的版本，那么我们应该做的第一件事情就是通过git init初始化。
    
    $ git init

git init只做了一件事情，那就是在项目的根目录下创建.git子目录来保存版本信息。

    $ ls .git
    branches/
    config
    description
    HEAD
    hooks/
    info/
    objects/
    refs/
    
上述命令显示了.git子目录中的内容。

## **2. 保存对象**
接下来让我们创建一个新的空文件test.txt。

    $ touch test.txt

然后把这个文件添加到Git代码库中，这一步将创建test.txt现有内容的一个副本。

    $ git hash-object -w test.txt

    e69de29bb2d1d6434b8b29ae775ad8c2e48c5391

在上述代码中，git hash-object命令将test.txt现有的内容压缩成二进制文件，并保存到Git中。该压缩文件叫做Git对象，保存在.git/objects目录中。
我们可以通过这个命令根据对象的文件名获取当前内容，并计算成SHA1 哈希（长度为40的字符串）。让我们看看下列新生成的Git对象文件。

    $ ls -R .git/objects
    .git/objects/e6:
    9de29bb2d1d6434b8b29ae775ad8c2e48c5391

如上述代码所示，.git/objects目录下又多出了一个子目录，而且这个子目录名是上述哈希值的前两个字符。在这个子目录下有一个文件，文件名是上述哈希值中其余的38个字符。
让我们再来看看文件内容。

    $ cat .git/objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391

上述代码输出的文件内容是一些二进制字符。你可能会问既然test.txt是空文件，又怎么会有这些内容呢？这是因为该二进制对象中还存储了一些元数据。
如果你想看看该文件原始的文本内容，那么应该使用git cat-file。

    $ git cat-file -p e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
    
因为原文件为空，所以上述命令什么都没有显示。现在我们往test.txt文件中写点东西。

   $ echo 'hello world' > test.txt

这个文件的内容已经改变了，所以你需要再次把它保存为Git对象。

    $ git hash-object -w test.txt
    3b18e512dba79e4c8300dd08aeb37f8e728b8dad
    
如上述代码所示，test.txt的哈希值已经随着文件内容的改变而发生了变化。同时还生成了新文件
git/objects/3b/18e512dba79e4c8300dd08aeb37f8e728b8dad。现在你可以看到这个文件的内容了。

    $ git cat-file -p 3b18e512dba79e4c8300dd08aeb37f8e728b8dad
    
    hello world

## **3. 更新索引**
当文件保存成二进制对象以后，你需要告诉Git哪个文件发生了变化。Git会在一个名叫“索引”（或阶段）的区域记录所有发生了变化的文件。然后等到所有的变更都结束后，将索引中的这些文件一起写入正式的版本历史记录中。

    $ git update-index --add --cacheinfo 100644 
    3b18e512dba79e4c8300dd08aeb37f8e728b8dad test.txt
    
上述命令记录了文件名test.txt、二进制对象名（哈希值）以及索引中文件的访问权限。
git ls-files命令可以显示索引中当前的内容。

    $ git ls-files --stage
    
    100644 3b18e512dba79e4c8300dd08aeb37f8e728b8dad 0   test.txt
    
上述代码显示索引中只有一个test.txt文件，还显示了该文件的二进制对象名和访问该文件的权限。如果你知道该二进制对象名，就可以查看.git/objects子目录中该文件的内容。
git status命令可以输出更多可读的结果。

    $ git status
    
    Changes to submit：
        The new file：   test.txt
        
上述代码显示索引中只有一个新文件test.txt，该文件正在等候写入版本的历史记录中。

## **4. git add命令**
针对每个文件执行上述两个步骤非常繁琐。所以Git提供了git add命令来简化这些操作。
    
    $ git add --all
    
上述命令相当于针对当前项目中所有发生了变化的文件执行上述两个步骤。

## **5. 提交（Commit）**
索引保存发生了变化的文件信息。等到修改完成，所有这些信息都会被写入版本的历史记录中，这相当于生成一个当前项目的快照。
项目的历史记录由不同时间点的项目快照组成。Git可以将项目恢复成任何一个快照。在Git中“快照”有一个专门的术语，即“提交”（commit）。所以生成快照也可以称之为完成提交。
下列所有“快照”的引用指的都是提交。

## **6. 完成提交**
首先，我们需要设置用户名和邮件地址。在你保存快照的时候，Git需要记录是谁执行的提交。
    $ git config user.name "username" 
    $ git config user.email "Email address"

接下来，保存现有的目录结构。在本文的前面我们讨论了保存对象只会保存一个文件，并不会记录文件之间的目录结构。
git write-tree命令可以根据当前目录结构生成一个Git对象。

    $ git write-tree

    c3b8bb102afeca86037d5b5dd89ceeb0090eae9d
    
在上述代码中，目录结构保存成了二进制对象，而对象的名字是哈希值。它也保存在.git/objects目录中。
让我们来看看该文件的内容。

    $ git cat-file -p c3b8bb102afeca86037d5b5dd89ceeb0090eae9d
    
    100644 blob 3b18e512dba79e4c8300dd08aeb37f8e728b8dad    test.txt
    
可以看到，当前目录中只有一个文件test.txt。
这个所谓的快照就是保存当前的目录结构，以及每个文件相对应的二进制对象。之前的操作已经保存了文件结构，所以现在你需要把这个目录结构和一些元数据一起写入版本的历史记录中。
git commit-tree可以将目录树对象写入到版本的历史记录中。

    $ echo "first commit" | git commit-tree c3b8bb102afeca86037d5b5dd89ceeb0090eae9d
    
    c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
    
在上述代码中，在提交时你需要提供提交的描述，而且你可以通过echo "first commit"提供提交描述。git commit-tree命令会根据元数据以及目录树生成一个Git对象。现在，让我们来看看该对象的内容。

    $ git cat-file -p c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
    
    tree c3b8bb102afeca86037d5b5dd89ceeb0090eae9d
    author jam  1538889134 +0800
    committer jam  1538889134 +0800
    
    first commit

在上述代码中，第一行输出是对应于该快照的目录树对象，而第二行和第三行是有关作者和提交者的信息，最后一行内容是提交的描述。
通过git log命令我们还可以查看某个快照的信息。

    $ git log --stat c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
    
    commit c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
    Author: jam 
    Date:   Sun Oct 7 13:12:14 2018 +0800
    
        first commit
    
     test.txt | 1 +
     1 file changed, 1 insertion(+)
     
## **7. git commit命令.**
Git提供了git commit来简化上述提交操作。在保存到索引后，你只需要执行git commit命令，就可以同时提交目录结构和描述，并生成快照。

    $ git commit -m "first commit"
    
另外，还有两个命令也非常实用。
通过git checkout命令，我们可以切换到某个快照。

    $ git checkout c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
    
通过git show命令，我们可以显示某个快照的所有代码变更。

    $ git show c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
    
## **8. 分支（branch）**
然而，如果你使用git log命令来查看整个版本的历史记录时，却无法看到刚刚生成的快照。

    $ git log
    
上述命令输出为空。这是为什么？这个快照刚刚不是写入到历史记录中了吗？



真相是：git log命令只可以显示当前分支上的变化。尽管我们已经提交了这个快照，但是还没有记录这个快照属于哪个分支。



分支是快照的指针，分支的名字就是该指针的名字。虽然哈希值不可读，但是分支允许用户给快照起别名。另外，分支还会自动更新，如果当前分支是一个新的快照，那么这个指针会自动指向它。例如，主分支（master branch）有一个名为master的指针指向主分支当前的快照。



用户可以为任何快照创建新指针。例如，如果你想创建一个新的fix-typo分支，那么只需创建一个名为fix-typo的指针，并指向一个快照。因此，在Git中创建一个新分支非常容易，而且开销非常低。



Git有一个特殊的指针HEAD，它始终指向当前分支中最新的那个快照。另外，Git还提供了快捷方式。例如，HEAD^指向HEAD之前的快照（父节点），而HEAD~6指向HEAD之前的第六个快照。



每个分支的指针都是一个文本文件，存储在.git/refs/heads/目录中。文件的内容是它指向的快照的二进制文件名（哈希值）。

## **9. 更新分支**
下面我们将演示如何更新分支。首先，修改test.txt。

    $ echo "hello world again" > test.txt
    
然后保存二进制对象。

    $ git hash-object -w test.txt
    
    c90c5155ccd6661aed956510f5bd57828eec9ddb
    
接下来，将该对象写入索引，并保存目录结构。

    $ git update-index test.txt
    $ git write-tree
    
    1552fd52bc14497c11313aa91547255c95728f37
    
最后，提交目录结构，并生成一个快照。

    $ echo "second commit" | git commit-tree 1552fd52bc14497c11313aa91547255c95728f37 -p c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
    
    785f188674ef3c6ddc5b516307884e1d551f53ca
    
在上述代码中，我们可以通过git commit-tree命令的参数-p来指定父节点，即以哪个快照为基础。
下面我们把快照的哈希值写入到.git/refs/heads/master文件中，并让master指针指向该快照。

    $ echo 785f188674ef3c6ddc5b516307884e1d551f53ca > .git/refs/heads/master
    
现在，通过git log命令你可以看到两个快照了。

    $ git log
    
    commit 785f188674ef3c6ddc5b516307884e1d551f53ca (HEAD -> master)
    Author: jam 
    Date:   Sun Oct 7 13:38:00 2018 +0800
    
        second commit
    
    commit c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa
    Author: jam 
    Date:   Sun Oct 7 13:12:14 2018 +0800
    
        first commit
        
git log命令的运行过程大致如下：



找到HEAD指针对应的分支。在上述示例中为master。



找到master指针指向的快照。在上述示例中为785f188674ef3c6ddc5b516307884e1d551f53ca。



找到父节点（即前一个快照）c9053865e9dff393fd2f7a92a18f9bd7f2caa7fa。

等等,最后显示当前分支中所有的快照。



另外，上述我们曾提到分支指针是动态的，下述三个命令会自动覆盖分支指针。



Git commit：当前分支的指针将移动到新创建的快照上。



Git pull：在当前分支和远程分支合并后，指针会指向新创建的快照。



Git reset [commit_sha]：当前分支的指针将被复位到某个指定的快照上。