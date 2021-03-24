# Homebrew命令大全

## 更新Homebrew自身

- brew update

## 使用Homebrew安装/卸载软件

- brew install/uninstall xxx

## 查询软件

- brew search xxx

## 查看所有安装软件包

- brew list

## 查看某个软件包的信息

- brew info xxx

## 显示有更新的软件包

- brew outdated

## 更新软件包

- brew upgrade - 更新所有
- brew upgrade xxx - 更新xxx软件包

## 卸载旧的软件包

- brew cleanup xxx - 卸载指定软件的老版本
- brew cleanup - 卸载所有软件包的老版本
- brew cleanup -n - 查看哪些软件包要被清除 

## 检查Homebrew的环境

- brew doctor - 检查系统的潜在问题
- brew config - 查看brew的全局配置

## Homebrew服务命令大全

- brew services list - 显示所有服务
- brew services run xxx - 运行xxx或所有服务，但不开机时自启动
- brew services start xxx - 开启xxx服务并开机时自启动
- brew services stop xxx - 停止xxx服务并关闭开机自启动服务
- brew services restart xxx - 重启xxx服务并开机时自启动