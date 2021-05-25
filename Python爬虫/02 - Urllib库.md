# Urllib库

> 本笔记写于2020年2月1日。Python版本为3.7.4，编辑器是VS code。
>
> 参考资料有：
>
> 1. B站视频[av44518113](https://www.bilibili.com/video/av44518113)
> 2. [崔庆才著《Python 3网络爬虫开发实战》](https://item.jd.com/12333540.html)
> 3. Python官方文档
>
> PS：如果笔记中有任何错误，欢迎在评论中指出，我会及时回复并修改，谢谢

## 基本概述

urllib库是Python内置的HTTP请求库，也是所有请求库的基础。其一共包括4个模块：

- urllib.request - 请求模块，用来构造HTTP请求
- urllib.error - 异常处理模块，用来处理请求错误
- urllib.parse - URL工具模块，提供了许多关于URL构造和处理的方法
- urllib.robotparser - robots.txt解析模块，识别网站的robots.txt文件，但一般很少用

## 发送请求

`urllib.request`是请求模块，通过该模块可以很方便的实现请求的发送并得到响应。

### urlopen方法

`urlopen()`函数是该模块最基本的请求函数，其一共包含7个参数。

|   参数    |                             作用                             | 默认值 |
| :-------: | :----------------------------------------------------------: | :----: |
|    url    |                 要请求的URL或一个Request对象                 |   -    |
|   data    | 需要发送给服务器的其他数据，同时将请求方式变为POST。数据需要使用`bytes()`方法转换为字节流格式，此时的返回内容同样是字节流格式 |  None  |
|  timeout  |                    设置超时时间，单位为秒                    |  None  |
|  cafile   |            3.6版本被废除，但方法中仍然保留该参数             |  None  |
|  capath   |            3.6版本被废除，但方法中仍然保留该参数             |  None  |
| cadefalut |            3.6版本被废除，但方法中仍然保留该参数             | False  |
|  context  |      ssl.SSLContext类型对象，进行SSL设置，用于HTTPS请求      |  None  |

下面通过例子来更好的认识这个函数

```python
# -*- coding: utf-8 -*-
import urllib.request

response = urllib.request.urlopen('http://www.baidu.com')
print(response.status)
```

这里要非常注意的是现在请求的是`http://www.baidu.com`这个页面，如果要请求`https://www.baidu.com`页面的话，上面的例子是会报错的。这主要是因为当请求`https://www.baidu.com`页面时会进行SSL校验，这个问题的正式解决方法比较复杂，如果有机会笔者会单独开一篇博客来讲解。现在可以简单的使用以下例子来爬取：

```python
# -*- coding: utf-8 -*-
import urllib.request
import ssl

# 导入ssl模块取消全局校验 
response = urllib.request.urlopen('https://www.baidu.com', context=ssl.SSLContext())
print(response.status)
```

通过上面的例子可以将百度首页的源代码爬取下来，我们可以使用`type()`函数来查看一下函数的返回值类型，发现是`http.client.HTTPResponse`类型。该类型是关于HTTP响应的包装，其中提供了大量的方法和属性来访问响应头和响应体。详细内容可以查看下面的补充。

下面通过示例来实验一下data参数：

```python
# -*- coding: utf-8 -*-
import urllib.request
import urllib.parse

data = bytes(urllib.parse.urlencode({'word': 'hello'}), encoding='utf-8')
response = urllib.request.urlopen('http://www.httpbin.org/post', data=data)
print(response.read())
```

结果为：

![ch03_data](/Users/wei/Github/Blog.Picture/Python/Spider/blog2/data.png)

这里请求的网站为`http://www.httpbin.org`，这是一个专门的HTTP测试网站。这里由于为`urlopen()`函数传递了data参数，因此请求形式变为POST形式，因此请求`/post`页面。使用`urllib.parse`模块来构建data参数，并通过`bytes()`方法将其转为字节流格式，编码为UTF-8编码。可以查看结果，我们构建的数据出现在了第一行的form字段中，并且结果开头的b表明了返回的结果也是字节流格式。

### Request类

`urlopen()`方法可以完成一个基本的请求发起。但现如今爬虫环境的泛滥，使得各网站都使用了一定的反爬虫机制，因此`urlopen()`方法提供的参数选项已经无法支撑一个完整的爬虫过程了。因此，Python的request库中提供了Request类，通过该类可以构建一个更为完整的请求。

> 注意：Request类只是构建了一个完整的请求对象，但无法进行发送请求，发送请求仍然还需要`urlopen()`方法的协助

该类的构造函数共有以下几个参数：

|      参数       |                             作用                             |                 默认值                  |
| :-------------: | :----------------------------------------------------------: | :-------------------------------------: |
|       url       |                         要请求的URL                          |                    -                    |
|      data       | 需要发送给服务器的其他数据。数据需要使用`bytes()`方法转换为字节流格式，此时的返回内容同样是字节流格式 |                  None                   |
|     headers     |              请求的请求头，是一个字典类型的数据              |                   {}                    |
| origin_req_host |                    请求方的请求名或IP地址                    |                  None                   |
|  unverifiable   |                      是否该请求无法验证                      |                  False                  |
|     method      |                 请求的方法，一般使用默认即可                 | 如果data的值为None，则为GET，否则为POST |

下面通过示例来验证一下这个类的使用

```python
# -*- coding: utf-8 -*-
import urllib.request as req
import urllib.parse as parse

url = 'http://httpbin.org/post'
headers = {
    'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_2) AppleWebKit/537.36 (KHTML, like Gecko) ' + 				  'Chrome/79.0.3945.130 Safari/537.36',
    'Host': 'httpbin.org'
}
dict = {
    'name': 'Wei'
}
data = bytes(parse.urlencode(dict), encoding='utf-8')
request = req.Request(url=url, data=data, headers=headers)
response = req.urlopen(request)
print(response.read().decode('utf-8'))
```

结果如下：

![ch03_Request](/Users/wei/Github/Blog.Picture/Python/Spider/blog2/request.png)

继续请求`https://httpbin.org/post`的页面，但这次没有直接在`urlopen()`方法中输入，而是通过构建一个`Request`对象来完成请求。

## 高级用法

上面是`urllib.request`模块中的最基本也是最重要的两个内容，但远不能说明请求模块的强大。请求模块还带有处理授权验证、重定向、Cookies等一系列的高级操作。

### Handler类

Handler类提供了非常强大的功能。通过Handler类可以完成几乎HTTP请求中的所有操作。Handler类并不是一个类，在`urllib.request`模块中共提供了22个Handler类，每一个Handler类都有着自己的作用~~（虽然大部分都用不到）~~。下面一起来看一下常用的几种Handler。

#### BaseHandler类

所有Handler类的基类，本身没有任何的作用，我们在爬虫的过程中也不会使用到它。这个类其实就是为了使用多态这一特性，配合下面介绍的`build_opener()`方法和OpenerDirector类来使用，为发送请求提供便利。

#### OpenerDirector类

在介绍Handler类之前，我们首先要介绍一下OpenerDirector类。这个类是一个非常重要的类。之前介绍的`urlopen()`方法的内部实现就是一个OpenDirector类。如果我们要使用Handler类来处理HTTP请求的话，上面介绍的`urloen()`方法就不再适用了，因为它本身就是一个Handler类的高级抽象。我们要完成更高级的请求，就需要深入底层去进行配置。OpenerDirector类常用的方法有：

|    方法     | 作用                                                         |
| :---------: | ------------------------------------------------------------ |
| add_handler | 传入一个BaseHandler类对象                                    |
|    open     | 发送请求，该方法的参数、返回值、异常等全部都与`urlopen()`方法相同 |

除了OpenerDirector类的方法外，我们也使用`build_opener()`方法来返回一个OpenerDirector类。该方法接受一个BaseHandler类对象，并返回一个OpenerDirector类。

#### ProxyHandler类

处理代理的Handler类。在做爬虫，尤其是大规模爬虫的时候，代理是免不了的。

```python

```

#### HTTPPasswordMgr系列类

#### 其他Handler类

其他Handler类就都没啥用了，反正我爬虫的时候是不会用这些类，这里只是记录一些。详情看下表。

|   Handler类   |                            作用                            |
| :-----------: | :--------------------------------------------------------: |
|  HTTPHandler  |   用于处理HTTP对应的URL。`urlopen()`方法的内部实现所需类   |
| HTTPSHandler  |  用于处理HTTPS对应的URL。`urlopen()`方法的内部实现所需类   |
|  DataHandler  |   用于处理Data对应的URL。`urlopen()`方法的内部实现所需类   |
|  FileHandler  | 用于处理本地文件对应的URL。`urlopen()`方法的内部实现所需类 |
|  FTPHandler   | 用于处理FTP协议对应的URL。`urlopen()`方法的内部实现所需类  |
| UnKnowHandler | 用于处理未知协议对应的URL。`urlopen()`方法的内部实现所需类 |

### Opener类

### 验证处理

### 代理处理

### Cookie处理

## 处理异常

第二节讲了如何发送请求，但实际上在现实生活中，请求经常会出现错误，如何处理错误是非常重要的一项工作。urllib库中定义了`error`模块专门来处理错误，其中包含了各种各样在`request`模块中可能出现的异常。一旦在请求过程中出现了问题，`request`模块便会抛出`error`模块中定义的异常。

`urllib.error`模块非常的简单，其中只有三个异常类，下面开始介绍

### URLError

从Python 3.3版本开始，URLError类的父类从IOError变为OSError。这个类是`error`模块的基类。

该类只有一个属性——**`reason`**，作用非常简单，返回错误原因。

## 解析URL

在爬虫过程中，对URL的处理是非常重要的一个部分。不论是手动请求还是自动请求，对URL的解析和处理都是必不可少的一个过程。因此，`urllib`库中专门提供了`parse`模块，这个模块的作用就是处理和解析URL。

在分析URL前，首先要知道一个标准的URL格式为：`scheme://netloc/path;params?query#fragment`，其中：

|   格式   | 说明                             |
| :------: | -------------------------------- |
|  scheme  | 协议类型                         |
|  netloc  | 服务器地址                       |
|   path   | 访问路径                         |
|  params  | 参数                             |
|  query   | 查询条件                         |
| fragment | 锚点，用于定位页面内部的下拉位置 |

### urlparse()和urlunparse()

`urlparse()`方法是最基本的分解方法，可以将一个URL解析为六部分。该方法一共有三个参数，但一般不会使用，这里就不再赘述了，有兴趣的可以查看Python的官方文档。

```python
from urllib import parse

url = 'http://www.baidu.com/index.html;user?id=5#comment'
result = parse.urlparse(url) # 直接将要解析URL传入
print(result)

# 结果为ParseResult(scheme='http', netloc='www.baidu.com', path='/index.html', params='user', query='id=5', fragment='comment')
```

上面的可以明显看出，`urlparse()`方法严格按照标准URL格式来分解的。

## 补充

### HTTPResponse

### 爬虫中常见的伪造请求头

在现如今的网络中，非常多的网站采用了反爬虫机制，部分机制会对请求头进行检测，如果发现爬虫会直接拒绝访问，所以要学会如何伪造请求头。

请求头有着非常的多内容，而在我们爬虫过程中，一般需要伪造的请求头中便是以下三个部分：

|   请求头   | 作用                                                         |
| :--------: | ------------------------------------------------------------ |
| User-Agent | 浏览器名称。如果不修改，该值默认为`python`，所以在现如今爬虫中，该请求头一定要修改 |
|  Referer   | 表明当前请求的是通过哪个URL发起的，如果不是特定URL，则拒绝请求 |
|   Cookie   | HTTP协议是无状态的，如果想要请求一些只有登录后才能发现的数据，就需要该请求头 |

一般来说，我在爬虫的过程中，只要上面弄好上面三个就基本没什么了。对于大部分网站来说，一般只需要伪造`User-Agent`就可以了，对于反爬虫机制强的网站来说，上面三个都要非常。如果伪造成功后还无法爬取成功，应该就不再是请求头的问题了。

### http.cookiejar

