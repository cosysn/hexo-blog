---
title: Hexo代码无法显示空行，并且行数和代码不对齐
date: 2017-07-20 00:02:55
tags:
---

## 0. 问题描述

使用hexo后显示代码块存在问题，会自动删除部分空行，以及行数和代码不对齐

比如我想显示下面的代码：

```c
#include <stdio.h>
 
int main(int argc, char *argv[])
{
	int a, b;

	printf("a+b", a+b);
	return 0;
}
```

但是实际的显示效果如下图所示：

![](code_incorrect.png)

我们可以看到空行被删除，导致行号和代码无法对其。

## 1. 问题分析

1. 首先看网页源码，看看对应的网页源码是否存在问题

   我们可以看到2和6这两个空行没有显示出来，而对应的html代码如下：

   ![](line_blank.png)

   上面标注的就是没有显示出来的行，可以看到虽然生成相应的div，但是其内容是空的。

   通过查询资料可以知道，如果div标签的内容为空的话，是无法在浏览器上显示出来的，详情见下面的链接：

   - [How to make a DIV with no content have a width?](https://stackoverflow.com/questions/4171286/how-to-make-a-div-with-no-content-have-a-width)
   - [HTML DIV element Disappears with no content](https://stackoverflow.com/questions/2114667/html-div-element-disappears-with-no-content)
   - [Div with background image and no content not displaying](https://stackoverflow.com/questions/6310808/div-with-background-image-and-no-content-not-displaying)

2. 上面知道了是因为空行生成的div的内容是空的导致浏览器无法显示出来，那么如果我们在写代码的时候，每个空行都加几个空格，保证div的内容不是空，是不是就可以将空行显示出来。

3. 经过测试，发现将空行上添加多个空格后，就能将空行正常显示出来了。

   ![](code_correct.png)

   而且生成的html代码中空行对应的div的内容也不是空的了。

   ![](line_blank_correct.png)

   虽然将每个空行添加了若干了空格暂时规避了该问题，但是那是治标不治本呀，定位到这里基本可以确定是hexo在处理markdown时没有处理这种情况，导致无法正常显示空行。

   下面我们去研究一下，hexo在处理markdown时除了什么问题导致这个现象。

4. 经过分析，发现生成代码片段的是在hexo-util/highlight.js文件中的highlightUtil函数，下面分析一下问题出现的原因：

   ```javascript
    var lines = data.value.split('\n');
     var numbers = '';
     var content = '';
     var result = '';
     var line;

     for (var i = 0, len = lines.length; i < len; i++) {
       line = lines[i];
       if (tab) line = replaceTabs(line, tab);
       numbers += '<div class="line">' + (firstLine + i) + '</div>';
       content += '<div class="line';
       content += (mark.indexOf(firstLine + i) !== -1) ? ' marked' : '';
       content += '">' + line + '</div>';
     }
   ```

   其处理流程如下：

   - 将获得的代码片段按照'\n'分割成若干个字符串，每行对应一个字符串；
   - 遍历所有行进行处理
     - 如果开启tab转空格选项，则调用replaceTabs将所有的tab转换为空格；
     - 生成行号对应的html片段
     - 生成代码对应的html片段

   从上面的处理流程可以看出，如果是空行的话，其会生内容为空的div。

5. 至此问题算是定位了。

## 2. 解决方案

经过上面的分析，我们已经知道了问题出现的原因，就是因为空行生成的div的内容为空。

那么我们在生成html代码时，可以判断如果是空行的话，可以在div中添加一个&nbsp，保证div的内容不为空。其代码的修改如下：

```javascript
var lines = data.value.split('\n');
  var numbers = '';
  var content = '';
  var result = '';
  var line;

  for (var i = 0, len = lines.length; i < len; i++) {
    line = lines[i];
    if (tab) line = replaceTabs(line, tab);
    numbers += '<div class="line">' + (firstLine + i) + '</div>';
    line = (line.length != 0) ? line : '&nbsp'; // 如果是空行的话，在div的内容添加个&nbsp
    content += '<div class="line';
    content += (mark.indexOf(firstLine + i) !== -1) ? ' marked' : '';
    content += '">' + line + '</div>';
  }
```

## 3. 问题验证

按上述解决方案修改之后，进行验证，发现问题已经解决了，可以正常显示代码。

如下图所示：

![](code_correct_verify.png)

而对应的代码片段如下：

![](line_blank_correct_verfy.png)

可以看到空行对应的div的内容全部被替换成了

## 4. 参考文献

1. [How to make a DIV with no content have a width?](https://stackoverflow.com/questions/4171286/how-to-make-a-div-with-no-content-have-a-width)
2. [HTML DIV element Disappears with no content](https://stackoverflow.com/questions/2114667/html-div-element-disappears-with-no-content)
3. [Div with background image and no content not displaying](https://stackoverflow.com/questions/6310808/div-with-background-image-and-no-content-not-displaying)
4. [hexo代码块前后空白行问题](https://leokongwq.github.io/2016/10/14/hexo-codeblock-multiblank.html)
5. [代码显示行是不是有点小问题呢？](https://github.com/forsigner/fexo/issues/46)
6. [显示代码块存在问题，会自动删除部分空行，以及行数和代码不对齐 #1](https://github.com/cosysn/hexo-blog/issues/1)