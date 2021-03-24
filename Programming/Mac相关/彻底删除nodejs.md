# Mac电脑彻底删除node.js

* 之前本来想搭建一个hexo来写博客的，但是最后还是放弃，老老实实就在博客园和CSDN写博文了，这里记录一下怎么在Mac电脑下彻底删除node.js的方法
* 下面这个方法是我结合了网上好几个方法综合在一起的，基本可以把所有与node有关的文件全部删除掉

1. 依次执行如下命令

    ```bash
    # 这里是卸载npm的
    sudo npm uninstall npm -g

    # 这里是用来删除node创建的各种文件夹
    sudo rm -rf /usr/local/lib/node
    sudo rm -rf /usr/local/lib/node_modules
    sudo rm -rf /var/db/receipts/org.nodejs.*
    sudo rm -rf /usr/local/include/node /Users/$USER/.npm*

    # 删除node命令
    sudo rm /usr/local/bin/node

    # 删除node的所有man手册
    sudo rm /usr/local/share/man/man1/node.1
    sudo rm /usr/local/share/man/man1/npm-*
    sudo rm /usr/local/share/man/man1/npm.1
    sudo rm /usr/local/share/man/man1/npx.1
    sudo rm /usr/local/share/man/man5/npm*
    sudo rm /usr/local/share/man/man5/package.json.5
    sudo rm /usr/local/share/man/man7/npm*

    # 这个命令也是删除一个node文件，但不知道这文件有什么用
    sudo rm /usr/local/lib/dtrace/node.d
    ```

2. node创建的各种文件夹确实是太多了，而且分布也十分乱，如果担心自己电脑中没有删除干净的话，可以运行下面两个命令在找一下

    ```Bash
    # 在/usr/local文件夹下查找以npm开头的文件
    find /usr/local -name 'npm*'
    # 在/usr/local文件夹下查找以node开头的文件
    find /usr/local -name 'node*'
    ```

    **PS: 删除的时候小心一些，不要把其他应用中以npm或node开头的文件删掉**

3. 删除结束后重启一下电脑，Mac会把与npm相关的命令重新设置一下

4. 执行命令

    ```Bash
    npm -v  
    // 结果应该是 -bash: npm: command not found
    node -v
    // 结果应该是 -bash: node: command not found
    ```
