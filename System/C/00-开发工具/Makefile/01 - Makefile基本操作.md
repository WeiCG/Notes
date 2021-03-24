# Makefile基本操作

> 本笔记写于2020年2月29日，系统为macOS Catalina 10.15.3，Make版本为3.81
>
> 记录一些makefile的基本操作
>
> 参考文献：Makefile官方文档

我使用的Make是我系统自带的Make，版本是3.81。如果想要最新版Make的人，可以去[官网](https://www.gnu.org/software/make/)上下载。

## 简单的Makefile

使用Make管理编译或链接的时候，必须要有一个makefile文件，该文件是Make的根本，这个文件里面告诉了Make如何编译以及链接程序。

makefile文件的基本内容如下：

```makefile
target ...: prerequisites ...
	recipe
	...
	...
```

下面一一来解释上面的语句：

### target

target，顾名思义，就是语句的目标，这个目标有三种情况：

1. executable file - 即可执行文件，也就是最终生成的目标
2. object file - 目标文件，生成可执行文件的中间产物
3. action name - 命令名称，详细信息查看之后

