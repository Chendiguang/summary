# Git常见的用法和难点记录

记录在工作中，git的一些常见用法。

## submodule

在一个项目中引入另外一个git项目，期望能及时更新对应的项目。这在golang项目里面尤其常见，最典型的例子时最近比较火热的k8s和etcd。

* 如何引入：

    ```bash
    cd yourproject
    # 这一步很关键，因为哪怕没有也不会报错，但是别人拉去项目时就更新不了子模块
    touch .gitmodules

    # 添加
    git submodule add git://github.com/chendiguang/haha.git haha

    # 查看
    git status
    ```

* 接收到一个包含子模块的项目时：

    ```bash
    # git clone xxxx.git
    # 初次下载时需要执行
    git submodule init

    git submodule update
    ```

## 分支操作

包括切换分支，与远程仓库的交互等
