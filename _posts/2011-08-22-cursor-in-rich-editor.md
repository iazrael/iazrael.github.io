---
layout: post
title:  "《富文本中的光标操作》总结"
date:   2011-08-22 21:29
categories: posts
---

#《富文本中的光标操作》总结

随着互联网的发展，用户对Web体验的要求越来越高。Web上常见的操作就是填写表单，传统的纯文本表现形式单一，不够直观。大部分UGC类型的网站都提供了富文本输入框给用户插入图片、音视频等丰富的内容。

在使用富文本时，因不同的浏览器接口不同，会遇到这样那样的兼容问题。其中的光标操作最为头疼，归纳起来，有以下五类光标问题：

## 一. 焦点移开后再聚焦回来光标位置不对


当用户输入内容中途，光标离开了输入框，之后聚焦回来，光标位置就跑到输入框的其实位置了。要继续在原来的位置输入就要用户重新定位，需要多一部操作，体验不佳。

要解决这个问题，我们可以在用户的光标离开输入框之间把当前光标位置保存起来，然后在聚焦回来时把光标位置还原。要注意的是还原光标位置在IE浏览器中有点麻烦，其代码如下：

	range.setEndPoint(‘EndToEnd’, lastRange);
	range.setEndPoint(‘StartToStart’,lastRange);
	range.select();
	
其中， setEndPoint方法可以移动当前的范围到指定的位置。这里的两个方法都是setEndPoint，并没有setStartPoint方法，其移动的位置根据第一个参数值来判断。要注意的是，EndToEnd的调用必须在StartToStart之前，因为range的起始位置是不能超过结束位置的，否则设置无效。

另外，IE浏览器还可以使用range的getBookmark和moveToBookmark方法来还原光标位置。但是经过测试，我发现如果在输入框以外的地方选中了内容，这个方法有20%左右的概率失效，因此不推荐使用。

## 二. 插入图片后光标位置不正确
在标准浏览器中插入图片有两种方法：`range.insertNode`和`document.execCommand`，其中execCommand方法只能插入特定的几个标签，不够灵活；而insertNode则可以插入任意dom节点（例如<br/>），因而选用insertNode方法。但也因此导致插入图片后光标定位在图片的前面。

要修复这个问题，可以在插入的时候，附加一个1px的img节点到节点后面，在插入之后把当前range设置到附加的节点后面，然后删除这个附加的img节点。设置range的代码如下：

	range.setEndAfter(lastNode);
	range.setStartAfter(lastNode);
	
这里有两个地方需要注意的：

1. 附加的节点选用img，是因为img是一个行内元素，同时又可以设置宽高（给普通的行内元素设置宽高是无效的）。因此插入这个节点后，用户是感觉不到的；
2. 创建dom节点可以使用`range.createContextualFragment`，该方法可以把传入的html字符串转换成dom节点返回。在调用insertNode把返回的dom节点插入时，实际上插入的是该dom节点的所有子节点，因此不会产生多余的容器。需要注意的是，IE9也支持标准的range，但是没有createContextualFragment方法，因此IE9还是建议使用其自由的range。

## 三. 删除选中的图片导致页面返回到上一页

在IE浏览器中，选中图片时，其实选中的range并不是TextRange，而是ControlRange。这时按下退格键（Backspace），会导致页面返回的上一页。要解决这个问题，可以监听keydown事件，如果是退格键且选中了ControlRange，就阻止其默认行为，并调用selection.clear()或者 `document.execCommand(‘Delete’, null, false)` 把选中图片删除。

## 四. 换行后行距很大
经过测试，发现在IE和Opera浏览器中，换行后生成的标签是<p>，而p标签默认是有行高的，就导致换行后的行距很大。我们可以通过监听keydown事件，把浏览器的默认换行行为阻止，然后插入一个<br/>标签来统一各个浏览器的表现。

这里有一些浏览器的兼容问题，如：

1. Window版的Opera浏览器中，无法通过阻止keydown事件的默认行为来禁止浏览器换行，可以换用keypress事件，其对阻止回车键有效；
2. Linux版的Firefox浏览器中，当开启了输入法后，输入框就不再触发出keydown事件，这里也需要换成监听keypress事件；
3. Chrome浏览器中，在行尾插入一个<br/>标签并不能使得光标移动到下一行，而如果插入<br/>加任意一个html标签都可以达到预期效果。因此这里可以重用插入图片的方法，使用一个附加的img节点，使光标换行，之后再删除它。删除节点是不能使用removeChild方法，必须用`document.execCommand(‘Delete’, null, false)`，否则光标会消失。

## 五. 过滤用户输入的内容后光标位置丢失

在使用富文本时，用和可能会从office或其他页面拷贝一段文本到输入框，而这些文本可能是附带样式的，导致输入框里的样式和布局错乱，用户体验相当差。另外，在使用用户输入的内容时，我们也必须对内容进行过滤检查，去除安全隐患。

过滤内容有两个方法：正则替换和遍历dom元素替换。正则替换后会导致光标位置完全无法还原，因此要做到用户插入文本并过滤后，光标跟在插入内容的后面，需要遍历dom节点。

需要注意的是：遍历时需要倒序进行，因为节点数组随着遍历而减少的，顺序的话数组下标方便处理；另外，倒序时，遇到的第一个非文本节点，就可以认为是用户插入内容的最后一个节点，把这个节点的文本的位置作为还原后的光标位置。最后，调用下面代码, 就可以把光标位置设置到插入内容的后面。

	selection.extend(cursorNode, cursorNode.length); 	selection.collapseToEnd();