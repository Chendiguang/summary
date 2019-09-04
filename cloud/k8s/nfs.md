# kubernetes使用nfs

* 创建一个nfs服务器(172.16.3.220)

    ```bash
    # 安装
    yum install rpcbind nfs-utils -y
    # 创建需要挂载到k8s集群中的目录
    mkdir -p /data/mnt

    # 添加共享目录配置 | 集群和nfs服务器所处的网段(172.16.3.0/24)
    echo "/data/mnt  172.16.3.0/24(rw,sync)" > /etc/exports

    # 添加到开机启动
    chkconfig rpcbind on
    chkconfig nfs on

    service nfs start
    service rpcbind start

    # 检查
    exportfs
    ```

* 在集群中安装

    ```bash
    yum install nfs-utils -y
    # 检查nfs的共享目录
    showmount -e 172.16.3.220

    # 挂载测试
    mount 172.16.3.220:/data/www-data /mnt

    # 此时在集群中可以看到nfs共享目录里面的内容
    ls /mnt
    ```
