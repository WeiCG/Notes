# Requests库

> 本笔记写于2020年2月8日。Python版本为3.7.4，编辑器为PyCharm Community 2019.3，对应requests库的版本为2.22.0
>
> 笔记参考文献：
>
> 1. B站视频[av44518113](https://www.bilibili.com/video/av44518113)
> 2. [崔庆才著《Python 3网络爬虫开发实战》](https://item.jd.com/12333540.html)
> 3. [Requests库官方文档](https://requests.readthedocs.io/en/master/)
>
> PS：如果笔记中有任何错误，欢迎在评论中指出，我会及时回复并修改，谢谢

在爬虫（二）Urilib库中，学习了`urllib`库的基本用法。但是，在`urllib`库确实有一些不好用的地方，比如其中关于cookie的处理、代理的使用等。因此，本节我们来学习一下`requests`库的用法。

## 基本用法

### 简单的GET请求和POST请求

比起`urllib`库的使用来说，`requests`库的使用非常的人性化。**如果我们想要发起一个GET请求，可以直接调用`requests`库中的`get()`方法；同理，发起POST请求，可以直接调用`post()`方法**。

下面简单看两个GET示例，因为这两个示例都比较简单，就放在一起：

```python
# coding: utf-8
import requests

# GET
response = requests.get('http://httpbin.org/get')
print(response.status_code)
print(response.text)

# POST
data = {
    'username': 'Mike'
}
response = requests.post('http://httpbin.org/post', data=data)
print(response.status_code)
print(response.text)
```

上面的示例分别向`http://httpbin.org/get`和`http://httpbin.org/post`两个网站发送了请求

GET请求结果如下：

![requests_get](/Users/wei/Github/Blog.Picture/Python/Spider/blog4/requests_get.png)

POST请求结果如下：

![requests_post](/Users/wei/Github/Blog.Picture/Python/Spider/blog4/requests_post.png)

可以看到，`requests`库的`get()`方法返回的是一个`Response`对象。该对象是`requests`库中自定义的一个底层类对应的对象，其中提供了大量有用的属性和方法，在我们平时的爬虫过程中都是必不可少的。

### 其他参数

在大多数的爬虫中，我们发起的请求总是要携带各种参数的。下面的表格中将介绍`get()`方法和`post()`方法常见的几个参数。

|    常见参数     | 作用                                                         |
| :-------------: | ------------------------------------------------------------ |
|     params      | `get()`方法。GET请求时要携带的数据。一般都是提供一个字典作为params参数的值 |
|      data       | `post()`方法。POST请求时要携带的数据。一般都是提供一个字典作为data参数的值 |
|      json       | `post()`方法。在POST请求中传输`json`数据                     |
|     headers     | 请求头参数                                                   |
|     cookies     | 要传递的Cookie。字典或`requests.cookies.RequestsCookieJar`对象 |
|     timeout     | 超时参数，单位秒                                             |
| allow_redirects | 是否允许重定向。默认为True                                   |
|     proxies     | 设置代理                                                     |
|     verify      | `bool`类型，表示是否校验SSL证书；`str`类型，指向CA证书的路径。默认为True |
|     stream      | 对于视频、图片、音频等是否立即下载。默认为False              |
|      auth       | 设置HTTP验证。元组类型                                       |

### 响应结果

#### Response类

`Response`类中有大量的属性和方法对我们解析结果有着非常大的用处，下面的表格将详细介绍这些常用的属性。

|  常用属性   | 作用                                                 |
| :---------: | ---------------------------------------------------- |
|   content   | 响应体返回的内容，字节码格式                         |
|   cookies   | 响应Cookie，`requests.cookies.RequestsCookieJar`对象 |
|  encoding   | text属性使用的编码，我们可以更改此编码               |
|   headers   | 响应头，可以直接查看                                 |
| status_code | 状态码                                               |
|     url     | 本次HTTP请求的最终URL                                |
|    text     | content属性解码后的内容，编码由encoding属性指定      |
|   history   | 如果存在重定向，该属性会按照请求顺序存储请求历史     |

#### 抓取下的数据

爬虫的抓取结果大致上可以分为三类，这三种抓取数据在`requests`库中都提供了非常好的解析方法。

##### `json`格式的数据

我们上面的两个示例实际上返回的都是`json`数据，如果想要直接解析的话，可以调用`Response`对象中的`json()`方法，可以将结果解析为字典格式。例如，我们将上面POST示例返回的结果解析为字典

##### ![requests_json](/Users/wei/Github/Blog.Picture/Python/Spider/blog4/requests_json.png)

需要注意的是，如果返回的数据不是`json`格式但调用了此方法，程序会报错

##### HTML页面

HTML页面的解析我们会在爬虫（五）中详细介绍

##### 二进制数据

主要是视频、音频、图片等文件。我们可以通过`Response`对象的`content`属性查看其二进制码，也可以通过文件IO的方式将其保存为对应的文件。

>  实质上，数据在网络中传输的格式都是二进制。我们看到的`json`格式和HTML页面都是`requests`库为我们底层进行了转换。我们都可以通过相同的操作查看其二进制码或将其保存为文件

## 高级设置

### Cookie设置

Cookie设置是一个非常重要的部分在爬虫中。如果我们是`urllib`库的话，这是一个比较复杂的写法，但`requests`库大大简化了这个流程。这里我们主要介绍一下响应过程中返回的Cookie，请求过程中构建的Cookie直接向`cookies`参数传递字典即可。

上文已经提到，响应过程中我们拿到的Cookie其实是一个`requests.cookies.RequestsCookieJar`对象。这个对象同样也是`requests`库自定义的一个Cookie类，其中提供了大量的属性和方法供我们解析Cookie。

| 属性或方法 | 作用                           |
| :--------: | ------------------------------ |
| get_dict() | 将Cookie转换为字典类型并返回   |
|  items()   | 将Cookie转换为键值对形式的元组 |
|   keys()   | 返回Cookie的名字列表           |
|  values()  | 返回Cookie的值列表             |

表中是一些解析响应Cookie时比较常用的函数。除此之外，该类还提供了一些构造Cookie时所需的方法，这里便不再赘述。

### Session对象的设置

在将Session对象之前，我们先来看一个示例：

```python
# coding: utf-8
import requests

# 第一次设置好了Cookie
response = requests.get('http://httpbin.org/cookies/set/username/Mike')
print(response.text)

# 第二次继续请求，期望获取Cookie
response = requests.get('http://httpbin.org/cookies')
print(response.text)
```

结果如下：

![Before_Session](/Users/wei/Github/Blog.Picture/Python/Spider/blog4/before_session.png)

出现这种情况主要是因为HTTP协议本身是无状态的，它是依靠Cookie来维持请求状态的。如果我们的请求没有添加相应Cookie的话，我们的请求实质上都是独立的请求，也就是上面的两个请求虽然是前后发出的，但却不存在任何关系。而如何在多次请求请求之间建立联系呢？就是使用Session对象。`requests`库为我们提供了Session对象，我们只需要在Session对象上进行各种请求操作即可，而有关多次请求之间的关联Cookie则全部交给Session来完成即可。下面在来看采用Session对象的示例：

```python
# coding: utf-8
import requests

session = requests.Session()
session.get('http://httpbin.org/cookies/set/username/Mike')
response = session.get('http://httpbin.org/cookies')
print(response.text)
```

结果如下，可以明显看到，第一次请求设置Cookie出现在了第二次请求中。

![After_Session](/Users/wei/Github/Blog.Picture/Python/Spider/blog4/after_session.png)

Session对象在平时的爬虫过程中多用于模拟登陆成功之后的爬取页面，这个在爬虫（十四）中我会详细的介绍。

### 代理设置

使用`requests`库设置代理非常简单，只需要在`get()`或`post()`方法的`proxies`参数指定即可。

下面来看一个例子：

```python
# coding: utf-8
import requests

proxies = {
    'http': 'http://1.196.177.218:9999'
}

response = requests.get('http://httpbin.org/ip', proxies=proxies)
```

结果如下：

![使用proxies](/Users/wei/Github/Blog.Picture/Python/Spider/blog4/requests_proxies.png)

需要注意的是，与`urllib`库不同的是，`requests`库在设置代理时，**必须在代理URL中设置协议头**，这是`requests`库官方文档中特别强调的一点。