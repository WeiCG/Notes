# npm全局权限设置 - Mac

> 在很多时候，即使使用了sudo命令来安装npm的全局模块包，仍然还会出现npm的权限问题

## 网络上的常见方法

* 目前网络上常见的方法都是修改npm安装路径的权限

通过使用命令

```Bash
 npm config get prefix
```

得到npm的路径，结果大部分都是/usr/local。然后，很多方法都会要求将该路径的权限修改为当前用户

即，使用命令

```Bash
 sudo chown -R $(whoami) /usr/local
```

或

```Bash
 sudo chown -R $(whoam) $(npm config get prefix)/{lib/node_modules,bin,share}
```

但这种方法在Mac OS达到10.12及之后便无效了，系统默认无法更改/usr/local文件夹的所有权，会提示: **chown: /usr/local: Operation not permitted**

---

## 官网给出的方法

然而实际上，node官网早已对这种情况作出了合理的修改方法，并且官网强烈建议用户不要使用root、sudo等方法覆盖权限。链接如下: [node官网给出的修改方法](https://docs.npmjs.com/resolving-eacces-permissions-errors-when-installing-packages-globally)

### 使用nvm重新安装node

nvm 即node version manager，这是node官网的推荐方法，使用nvm安装node时会自动申请各种权限，在之后的使用中就不会有权限问题了

安装方法如下: [node官网给出的安装方式](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)

### 改变npm的默认路径

第二个方法是我目前使用的方法，这也是不想重新安装node的用户可以采用的方法。可以将默认的全局安装路径修改到**当前用户的home目录**下

1. 新建一个全局安装的路径

    ```Bash
     mkdir ~/.npm-global
    ```

2. 配置npm使用新的路径

    ```Bash
     npm config set prefix ‘~/.npm-global’
    ```

3. 打开或新建~/.bash_profile文件，在末尾加入

    ```Bash
     export PATH=~/.npm-global/bin:$PATH
    ```

4. 更新系统环境变量

    ```Bash
     source ~/.bash_profile
    ```

* PS: 如果你不想去修改.bash_profile文件的话，你也可以使用如下命令

    ```Bash
     # 配置npm config的路径
     NPM_CONFIG_PREFIX=~/.npm-global
    ```

## 使用npx安装全局模块

同时，如果你的npm版本是在5.2及以上的时候，在安装npm全局模块时，node官方更推荐你使用npx命令。[npx的官方指南连接](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b)

* **注意: 上面那个网站可能需要翻墙，所以可以直接百度npx的使用方式**
