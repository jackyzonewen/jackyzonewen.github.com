---
layout: post
title: "MarkDown使用"
date: 2014-04-03 11:42
comments: true
categories: other
---

> <p>Markdown 语法的目标是：成为一种适用于网络的书写语言。Markdown不是想要取代 HTML，甚至也没有要和它相近，它的语法种类很少，只对应 HTML 标记的一小部分。不在 Markdown 涵盖范围之内的标签，都可以直接在文档里面用 HTML 撰写。不需要额外标注这是 HTML 或是 Markdown；只要直接加标签就可以了。

---

<h4> 1.标题 </h4>

使用`<h4> 形式 </h4>`或者`#### 标题`的形式来表示四级标题，总共支持6级标题。  
注：标题文字前后留一个空格，标准的mardDown语法

---

#### 2.列表 

无序列表写法  

- 列表1
  + 子列表1
  + 子列表2
  + 子列表3
- 列表2
- 列表3
- 列表4

有序列表的写法  

1. 列表1
2. 列表2
3. 列表3
4. 列表4

注意：-(*或者+)、1. 和列表之间保留一个字符的空格

---

#### 链接和图片

在 Markdown 中，插入链接不需要其他按钮，你只需要使用 `[显示文本](链接地址)` 或者在文章最后给上索引，`[显示文本][1]` ，例如：  

	[我的博客](http://jackyzonewen.github.io/)
	或者
	[我的博客][1]
	[1]:(http://jackyzonewen.github.io/)

在 Markdown 中，插入图片不需要其他按钮，你只需要使用 `![](图片链接地址)` 这样的语法即可，例如：  

	![](http://ww4.sinaimg.cn/bmiddle/aa397b7fjw1dzplsgpdw5j.jpg)
	
---

#### 引用

只需要在你希望引用的文字前面加上 > 就好了，例如：  

	> 一盏灯， 一片昏黄； 一简书， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。

效果如下:  
> 一盏灯， 一片昏黄； 一简书， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。

---

#### 粗体和斜体

Markdown 的粗体和斜体也非常简单，用两个 * 包含一段文本就是粗体的语法，用一个 * 包含一段文本就是斜体的语法。例如：  

*一盏灯*， 一片昏黄；**一简书**， 一杯淡茶。 守着那一份淡定， 品读属于自己的寂寞。 保持淡定， 才能欣赏到最美丽的风景！ 保持淡定， 人生从此不再寂寞。  

---

#### 代码

行开头空4格空格，或者一个Tab来表示程序代码，例如:

    //这里显示一些代码，在正文显示中会自动识别语言，进行代码染色，这是一段C#代码
    public class Blog
    {
         public int Id { get; set; }
         public string Subject { get; set; }
    }

---

#### 其它

	单个回车：空格
	连续回车：分段
	行尾连续两个空格：段内换行

---

介绍markdown的教程很多，提供几个供大家参考:

- [鲁塔弗：markdown 简明语法][5]  
- [图灵社区：怎样使用Markdown][6]  
- [简书：献给写作者的 Markdown 新手指南][7]  
- [官方文档(中文版)：Markdown 语法说明][8]  
- [用Markdown来书写你的博客][9]  


编辑工具:

- [简书][10]  
- [MaDe (Chrome插件)][11]  
- [dillinger][12]  
- [StackEdit][13] 
- [Cmd][14]

#### 参考教程：  

- [鲁塔弗：markdown 简明语法][5]  
- [图灵社区：怎样使用Markdown][6]  


[5]: http://lutaf.com/markdown-simple-usage.htm
[6]: http://www.ituring.com.cn/article/23
[7]: http://jianshu.io/p/q81RER
[8]: http://wowubuntu.com/markdown/#p
[9]: http://upwith.me/?p=503
[10]: http://jianshu.io/
[11]: https://chrome.google.com/webstore/detail/made/oknndfeeopgpibecfjljjfanledpbkog
[12]: http://dillinger.io/
[13]: https://stackedit.io/
[14]: http://www.zybuluo.com/mdeditor




