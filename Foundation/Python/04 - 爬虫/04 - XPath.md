# XPath

>本笔记写于2020年2月9日。Python版本为3.7.4，编辑器为PyCharm Communtiy 2019.3
>
>笔记参考文献：
>
>1. [崔庆才著《Python 3网络爬虫开发实战》](https://item.jd.com/12333540.html)
>2. [W3Schools的XPath教程](https://www.w3schools.com/xml/xpath_intro.asp)
>
>PS：如果笔记中有任何错误，欢迎在评论中指出，我会及时回复并修改，谢谢

上一节我们讲解了`requests`库的使用，目前我在爬虫的时候，一般常用的请求库也就是`urllib`库和`requests`库这两个。

下面我将开始学习解析库的使用。实际上，Python爬虫的解析HTML页面的方法非常的多，有lxml库、Beautiful Soup库、正则表达式等。一般来说，众多的解析库只学习一种便已经足够了，都差不多。万法会不如一法精吗。我目前使用的是lxml库搭配XPath的方式，效率很高，基本可以解析大部分的页面。实在解析不出来的就直接使用正则表达式。

因此，我接下来的三篇博客会分别介绍XPath语法、lxml库和正则表达式，其余的解析库就看情况吧。

## XPath概述

XPath，全称为XML Path Language，这是一门在XML文档中查找信息的语言。虽然是专门为XML设计的，但同样也适用于HTML文档的查询。XPath的功能非常强大，不仅提供了简单的路径选择表达式，非常容易就可以定位节点，同时还包含了超过200个内置函数，用于各种方面的处理。

## XPath术语

在详细讲述XPath语法之前，我们首先需要看一下XPath相关的术语。为了更方便的解释，我直接就把官方的示例给搬过来了。下面的术语都将围绕此示例展开。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<bookstore>
	<book>
        <title lang="en">Harry Potter</title>
        <author>J K. Rowling</author>
        <year>2005</year>
        <price>29.99</price>
    </book>
</bookstore>
```

### 节点(node)

在XPath中一共存在**元素(element)、属性(attribute)、文本(text)、命名空间(namespace)、处理指令(processing-instruction)、注释(comment)和文档(document)等**七种节点。XPath非常奇妙的就是它将要解析的HTML或XML文档看做了一棵节点树，而这颗节点树的根标签被称作根元素标签。

> 关于节点这部分的内容，我看得是一头雾水。因为在英文网站中，明确写出了*XML documents are treated as trees of nodes. The topmost element of the tree is called the root element*，而在中文网站却翻译为了*XML 文档是被作为节点树来对待的。树的根被称为文档节点或者根节点*。这个文档节点四个字我不知道是怎么出来的，当然也有可能是因为我本身只有很少的XML基础，所以我对这部分的理解不到位造成的。但节点这节确实的看得烦，如果以后有机会的话，我还会在回来补充一下这些。

我们以上面的示例为例，其中

```xml
<bookstore>  (根元素节点)
<autuor>J K. Rowling</autuor>  (元素节点)
lang="en"  (属性节点)
```

当然，对应的节点之间还存在着相互关系，包括父(Parent)、子(Children)、兄弟(Sibling)、祖(Ancestor)、后代(Descendant)，意思都很明确就不再赘述了。

### 原子值(Atomic value)

又是一个完全雾水的概念。原子值在W3Schools上的说法是*nodes with no children or parent*，即原子值是一种特殊的节点，其既无父也无子。但我一直搞不清楚这个无父无子指的是什么，官方的示例我也觉得没有明显表达出这个无父无子的状态。我就直接将原子值作为单纯的文本看待了。

下面是官网文档给出的示例：

```xml
J K. Rowling
"en"
```

## XPath语法

讲完了相关术语，我们便可以来看一下XPath语法了。首先来看一个示例，这也是W3Schools网站上的例子。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<bookstore>
    <book>
        <title lang="en">Harry Potter</title>
        <price>29.99</price>
    </book>
    <book>
        <title lang="en">Learning XML</title>
        <price>39.95</price>
    </book>
</bookstore>
```

#### 选择节点

既然将整个文档看作了一颗节点树，那么我们必要是要选择节点的。XPath提供了路径表达式来让我们选择节点。

路径表达式语法如下：

|  表达式  | 作用                                   |
| :------: | :------------------------------------- |
| nodename | 选择所有名称为*nodename*的节点         |
|    /     | 选择根节点（从当前节点选取直接子节点） |
|    //    | 从当前节点中选取所有匹配的节点         |
|    .     | 选中当前节点                           |
|    ..    | 选中当前节点的父节点                   |
|    @     | 选择属性节点                           |

看似非常美好的语法，但其实全部都是坑。下面我细细给你们列。

1. 首先，nodename不能单独使用。比如我想直接使用`book`来选择上面示例中的两个book节点，然并卵。虽然网站上明确写明了*Selects all nodes with the name "nodename"*，但实际上如果直接使用nodename，是没有选择任何节点的，必须要在前面搭配相关的路径才能选择。有个例外就是选取根元素节点是可以直接选取的。而且中文网站上的翻译为：*选取此节点的所有子节点*。译者是初中没毕业吗？
2. 第二个语法`/`也是很坑的描述。什么叫做*Selects from the root node*。这个描述我觉得《Python 3网络爬虫开发实战》的描述还不错，就是**从当前节点选取直接子节点**



