---
layout: post
title: 【Idea】liveTemplate 方法注释之params,return参数自动获取
category: tools
tags: [tools]
---


# 问题解决

#### 第一步：打开设置

![这里写图片描述](https://img-blog.csdn.net/20180702202633215?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxcXEwMTk5MTgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 第二步：打开live templates

![这里写图片描述](https://img-blog.csdn.net/20180702202859562?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxcXEwMTk5MTgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
首先点击**Editor**下的**Live Templates**,再点击右上角的**加号**，然后选取**Template Group**

#### 第三步：创建一个自己的Template Group

在弹出的窗口中填入自己的Template Group名称（我填MyGroup）
![这里写图片描述](https://img-blog.csdn.net/20180702203244362?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxcXEwMTk5MTgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
点击OK后可以看到列表中多出一下MyGroup
![这里写图片描述](https://img-blog.csdn.net/20180702203359810?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxcXEwMTk5MTgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 第四步：打开Live Template编辑界面

![这里写图片描述](https://img-blog.csdn.net/20180702203539663?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxcXEwMTk5MTgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
选中刚创建的**MyGroup**,点击右上角**加号**，选择**Live Template**。

#### 第五步：编辑Live Template

接下来的内容别问为什么，照抄就是！
![这里写图片描述](https://img-blog.csdn.net/20180702203906735?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxcXEwMTk5MTgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

<code class=" hljs ruby">**
 *
 * @author : 作者名称
 * @date : $date$ $time$ 
$params$ $return$
 */</code>

![这里写图片描述](https://img-blog.csdn.net/20180702204010881?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxcXEwMTk5MTgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
params参数的内容

<code class=" hljs scilab">groovyScript("if(\"${_1}\".length() == 2) {return '';} else {def result=''; def params=\"${_1}\".replaceAll('[\\\\[|\\\\]|\\\\s]', '').split(',').toList();for(i = 0; i < params.size(); i++) {if(i<(params.size()-1)){result+=' * @param ' + params[i] + ' : ' + '\\n'}else{result+=' * @param ' + params[i] + ' : '}}; return result;}", methodParameters());</code> 

return参数的内容

<code class=" hljs smalltalk">groovyScript("def returnType = \"${_1}\"; def result = ' * @return : ' + returnType; return result;", methodReturnType());</code>

OK，保存设置，下面开始用！

# 开始用

![这里写图片描述](https://img-blog.csdn.net/20180702204412621?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxcXEwMTk5MTgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
我们将对上图的方法加注释
首先输入

<code class=" hljs tex">\add</code>

这时候IDEA会自动弹出提示如下
![这里写图片描述](https://img-blog.csdn.net/20180702204629284?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxcXEwMTk5MTgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
按下enter键
![这里写图片描述](https://img-blog.csdn.net/20180702204729303?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxcXEwMTk5MTgx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这样，方法注释就自动获取到了方法参数和返回值类型。


 
