# pip命令大全

新版Mac系统上安装了Python3，所以可以直接在系统Python上工作。同时，由于Python自3.4版本之后自带pip，因此，也不需要去主动安装pip了。但是，坑爹的地方来了，虽然新版Mac安装了Python3，但只放在了开发工具的目录下，也就是说还没有让系统默认使用Python3，pip调用时，默认使用的是Python2，因此，pip会提示警告，解决办法是使用`python -m pip xxx`的命令

## 升级pip

- pip install -U pip

## 帮助命令

- pip --help

## 列出以安装的包

- pip list

## 安装/卸载包

- pip install/uninstall xxx
- pip install xxx==版本号

## 升级包

- pip install -U xxx

## 显示包所在目录

- pip show -f xxx

## 查询可升级包

- pip list -o

## 搜索包

- pip search xxx