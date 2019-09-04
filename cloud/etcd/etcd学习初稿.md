# Etcd

* 入口函数

    ```go
    // main.go
    import "go.etcd.io/etcd/etcdmain"

    func main() {
        etcdmain.Main()
    }
    ```

* Main函数

    Main函数里面主要有做了三件事情：平台架构支持检查、corba命令行解释执行、启动etcd或代理

    ```go
    // package etcdmain
    // etcd.go

    func Main() {
    checkSupportArch()

    // ...
    // rootCmd.SetArgs(args)
    // rootCmd.Execute()
    startEtcdOrProxyV2()
    }
    ```

    接下来看一下startEtcdOrProxyV2里面做了些什么

    ```go
    ```
