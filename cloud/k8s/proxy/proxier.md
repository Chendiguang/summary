# ```Proxy```

kube-proxy, 首先了解一下大致的目录架构。每一个模块都分别在cmd和pkg目录下面实现分离后的代码。cmd中的每个模块是基于[cobra](https://github.com/spf13/cobra)实现的命令行功能，pkg是实现模块的核心代码。

[目录索引]

* [1.Cmd](#cmd)

  * [1.1 主入口函数](#cmd-main)

    * [1.1.1 代码](#cmd-code)

    * [1.1.2 其他](#cmd-other)

  * [1.2 app](#app)

    * [1.2.1 server.go](#app-server)

    * [1.2.2 server_other.go](#app-other)

    * [1.2.3 conntrack.go](#app-conntrack)

## <span id=cmd>```1. Cmd```</span>

这里查看的是cmd的代码架构。

### <span id=cmd-main>1.1 主入口函数</span>

代码位于$gopath/k8s.io/kubernetes/cmd/kube-proxy/proxy.go, 主要需要了解的就是```cobra和promethus```。

* <span id=cmd-code>1.1.1 代码</span>

    ```go
    func main() {
    rand.Seed(time.Now().UnixNano())

    command := app.NewProxyCommand()

    // TODO: once we switch everything over to Cobra commands, we can go back to calling
    // utilflag.InitFlags() (by removing its pflag.Parse() call). For now, we have to set the
    // normalize func and add the go flag set by hand.
    pflag.CommandLine.SetNormalizeFunc(cliflag.WordSepNormalizeFunc)
    pflag.CommandLine.AddGoFlagSet(goflag.CommandLine)
    // utilflag.InitFlags()
    logs.InitLogs()
    defer logs.FlushLogs()

    if err := command.Execute(); err != nil {
        fmt.Fprintf(os.Stderr, "error: %v\n", err)
        os.Exit(1)
    }
    }
    ```

* <span id=cmd-other>1.1.2 此外</span>

    关注一下这两句代码，注册了promethues的监控功能

    ```go
    import (
    //....
    _ "k8s.io/kubernetes/pkg/client/metrics/prometheus" // for client metric registration
    _ "k8s.io/kubernetes/pkg/version/prometheus"        // for version metric registration
    )
    ```

    进入到cmd/kube-proxy目录，可以直接执行go build -v来编译获得相应的二进制执行文件，当然也可以添加额外的命令参数

### <span id=app>1.2 app</span>

main函数文件引入了```"k8s.io/kubernetes/cmd/kube-proxy/app"```这个包，这就是kube-proxy 实现cmd的全部代码。们只需要关注其中这几个文件：```conntrack.go, server_other.go, server.go```

* <span id=app-server>1.2.1 server.go</span>

    根据main() 函数调用的app.NewProxyCommand()我们首先进入server.go查看。下面不是具体代码，而是提取出来的主干代码

    ```go
    func NewProxyCommand() *cobra.Command {
        opts := NewOptions()
        cmd := &cobra.Command{}
        opts.config, err = opts.ApplyDefaults(opts.config)

        return cmd
    }
    ```

    详细的代码就不罗列了，主要的作用就是根据配置文件初始化一个配置结构体```opts := NewOptions()```，然后利用cobra的典型应用构建一个命令```cmd := &cobra.Command{}```。所以这里需要重点了解NewOptions()这个函数生成的结构体的对象，还有它的方法。

    1. Options结构体

        ```go
        type Options struct {
            ConfigFile string // proxy server配置文件的位置

            WriteConfigTo string  // 默认配置的保存位置

            CleanupAndExit bool // 是否需要清除iptables rules，并且退出

            CleanupIPVS bool // 在退出前是否清除ipvs rules

            WindowsService bool // 运行在windows时配置

            config *kubeproxyconfig.KubeProxyConfiguration // proxy server 配置对象

            watcher filesystem.FSWatcher // 利用一个第三方库FSWatcher来监控配置文件的变化

            proxyServer proxyRun // 运行proxy server的接口

            errCh chan error // errCh 接收错误信息的channel

            // master is used to override the kubeconfig's URL to the apiserver.
            master string
            // healthzPort is the port to be used by the healthz server.
            healthzPort int32
            // metricsPort is the port to be used by the metrics server.
            metricsPort int32

            scheme *runtime.Scheme
            codecs serializer.CodecFactory  // 编解码器

            // hostnameOverride, if set from the command line flag, takes precedence over the `HostnameOverride` value from the config file
            hostnameOverride string
        }
        ```

    2. Options的几个方法

        这几个是完成配置项的前提操作，主要是解析配置文件，验证配置信息，初始化Options对象

        ```go
        // AddFlags adds flags to fs and binds them to options.
        func (o *Options) AddFlags(fs *pflag.FlagSet) {}

        // NewOptions returns initialized Options
        func NewOptions() *Options {}

        // Complete completes all the required options.
        func (o *Options) Complete() error {}
        ```

        多种导入配置文件的方式

        ```go
        // 当len(o.ConfigFile) > 0时被调用，事实上最终调用了loadConfig
        func (o *Options) loadConfigFromFile(file string) (*kubeproxyconfig.KubeProxyConfiguration, error) {}

        func (o *Options) loadConfig(data []byte) (*kubeproxyconfig.KubeProxyConfiguration, error) {}
        ```

        验证和保证配置文件异常时，可以使用默认配置进行初始化。

        ```go
        // Validate validates all the required options.
        func (o *Options) Validate(args []string) error {}

        // ApplyDefaults applies the default values to Options.
        func (o *Options) ApplyDefaults(in *kubeproxyconfig.KubeProxyConfiguration) (*kubeproxyconfig.KubeProxyConfiguration, error) {}
        ```

        接下来是，注册事件监听handler，主要利用了FsnotifyWatcher来监听配置文件，channel通知事件信息

        ```go
        // Creates a new filesystem watcher and adds watches for the config file.
        func (o *Options) initWatcher() error {
            fswatcher := filesystem.NewFsnotifyWatcher()
            err := fswatcher.Init(o.eventHandler, o.errorHandler)
            if err != nil {
                return err
            }
            err = fswatcher.AddWatch(o.ConfigFile)
            if err != nil {
                return err
            }
            o.watcher = fswatcher
            return nil
        }

        func (o *Options) eventHandler(ent fsnotify.Event) {
            eventOpIs := func(Op fsnotify.Op) bool {
                return ent.Op&Op == Op
            }
            if eventOpIs(fsnotify.Write) || eventOpIs(fsnotify.Rename) {
                // error out when ConfigFile is updated
                o.errCh <- fmt.Errorf("content of the proxy server's configuration file was updated")
            }
            o.errCh <- nil
        }

        func (o *Options) errorHandler(err error) {
            o.errCh <- err
        }
        ```

    3. Proxy server的接口

        下面的代码片段显然得知，```ProxyServer实现了proxyRun接口```

        ```go
        // proxyRun defines the interface to run a specified ProxyServer
        type proxyRun interface {
            Run() error
            CleanupAndExit() error
        }

        func (s *ProxyServer) Run() error {}

        // CleanupAndExit remove iptables rules and exit if success return nil
        func (s *ProxyServer) CleanupAndExit() error {}
        ```

    4. 运行代码

        前面提到，架构是基于cobra框架的，所以这一级的命令调用如下，上面的实现都是为了如下代码做准备。

        ```go
        // NewProxyCommand creates a *cobra.Command object with default parameters
        func NewProxyCommand() *cobra.Command {
            opts := NewOptions()

            cmd := &cobra.Command{
                Use: "kube-proxy",
                Long: `The Kubernetes network proxy runs on each node. This
        reflects services as defined in the Kubernetes API on each node and can do simple
        TCP, UDP, and SCTP stream forwarding or round robin TCP, UDP, and SCTP forwarding across a set of backends.
        Service cluster IPs and ports are currently found through Docker-links-compatible
        environment variables specifying ports opened by the service proxy. There is an optional
        addon that provides cluster DNS for these cluster IPs. The user must create a service
        with the apiserver API to configure the proxy.`,
                Run: func(cmd *cobra.Command, args []string) {
                    verflag.PrintAndExitIfRequested()
                    utilflag.PrintFlags(cmd.Flags())

                    if err := initForOS(opts.WindowsService); err != nil {
                        klog.Fatalf("failed OS init: %v", err)
                    }
                    // 配置信息的准备过程
                    if err := opts.Complete(); err != nil {
                        klog.Fatalf("failed complete: %v", err)
                    }
                    if err := opts.Validate(args); err != nil {
                        klog.Fatalf("failed validate: %v", err)
                    }
                    klog.Fatal(opts.Run())
                },
            }

            var err error
            opts.config, err = opts.ApplyDefaults(opts.config)
            if err != nil {
                klog.Fatalf("unable to create flag defaults: %v", err)
            }

            opts.AddFlags(cmd.Flags())

            // TODO handle error
            cmd.MarkFlagFilename("config", "yaml", "yml", "json")

            return cmd
        }
        ```

        其中我们重点需要关注的是，opts.Run()这个函数，最终的调用是proxyServer.Run()，具体查看它的接口实现。

        ```go
        // Run runs the specified ProxyServer.
        func (o *Options) Run() error {
            defer close(o.errCh)
            if len(o.WriteConfigTo) > 0 {
                return o.writeConfigFile()
            }

            proxyServer, err := NewProxyServer(o)
            if err != nil {
                return err
            }

            if o.CleanupAndExit {
                return proxyServer.CleanupAndExit()
            }

            o.proxyServer = proxyServer
            return o.runLoop()
        }

        // runLoop will watch on the update change of the proxy server's configuration file.
        // Return an error when updated
        func (o *Options) runLoop() error {
            if o.watcher != nil {
                o.watcher.Run()
            }

            // run the proxy in goroutine
            go func() {
                err := o.proxyServer.Run()
                o.errCh <- err
            }()

            for {
                err := <-o.errCh
                if err != nil {
                    return err
                }
            }
        }
        ```

        其中，在整个调用栈里，最终执行

        ```go
        func (s *ProxyServer) Run() error {}
        ```

* <span id=app-other>1.2.2 server_other.go</span>

    linux平台下ProxyServer的部分在这个文件里面实现。

    设置根据环境和配置获取代理模式ProxyMode

    ```go
        func getProxyMode(proxyMode string, iptver iptables.Versioner, khandle ipvs.KernelHandler, ipsetver ipvs.IPSetVersioner, kcompat iptables.KernelCompatTester) string {
        switch proxyMode {
        case proxyModeUserspace:
            return proxyModeUserspace
        case proxyModeIPTables:
            return tryIPTablesProxy(iptver, kcompat)
        case proxyModeIPVS:
            return tryIPVSProxy(iptver, khandle, ipsetver, kcompat)
        }
        klog.Warningf("Flag proxy-mode=%q unknown, assuming iptables proxy", proxyMode)
        return tryIPTablesProxy(iptver, kcompat)
    }
    ```

    再结合tryIPTablesProxy()和tryIPVSProxy() 的代码可知：优先根据配置文件的设置进入不同的ProxyMode选取函数，
    Userspace是直接返回，其他两种的根据os是否支持来确定是否继续按照一下优先级往下走: ```IPVS-->IPTables-->Userspace```

    在iptables模式下，kube-proxy使用了iptables的filter表和nat表，并且对iptables的链进行了扩充，自定义了KUBE-SERVICES、KUBE-NODEPORTS、KUBE-POSTROUTING和KUBE-MARK-MASQ四个链，另外还新增了以“KUBE-SVC-”和“KUBE-SEP-”开头的数个链

* <span id=app-conntrack>1.2.3 conntrack.go</span>

    整个文件实现的功能是修改linux下面的链路跟踪的参数。查看这些参数可以参考下面的命令

    ```bash
    lsmod |grep nf_conntrack

    # 查找所有，再过滤
    sysctl -a |grep nf_conntrack

    # 更具体的用法
    sysctl net.netfilter.nf_conntrack_max
    # 事实上是在文件里保存的
     cat /proc/sys/net/netfilter/nf_conntrack_max

    dmesg | grep conntrack
    ```

    Conntracker接口

    ```go
    type Conntracker interface {
        // SetMax adjusts nf_conntrack_max.
        SetMax(max int) error
        // SetTCPEstablishedTimeout adjusts nf_conntrack_tcp_timeout_established.
        SetTCPEstablishedTimeout(seconds int) error
        // SetTCPCloseWaitTimeout nf_conntrack_tcp_timeout_close_wait.
        SetTCPCloseWaitTimeout(seconds int) error
    }
    ```

    相关的调用代码

    ```go
    func (s *ProxyServer) Run() error {
        // ...
        // Tune conntrack, if requested
        // Conntracker is always nil for windows
        if s.Conntracker != nil {
            max, err := getConntrackMax(s.ConntrackConfiguration)

            // ...
            if s.ConntrackConfiguration.TCPEstablishedTimeout != nil && s.ConntrackConfiguration.TCPEstablishedTimeout.Duration > 0 {
               // ...
            }

            if s.ConntrackConfiguration.TCPCloseWaitTimeout != nil && s.ConntrackConfiguration.TCPCloseWaitTimeout.Duration > 0 {
                // ...
            }
        }
    }
    ```

## 总结

k8s通过在目标node的nat表中的PREROUTING(路由前)和POSTROUTING(路由后)链中创建一系列自定义链 （这些自定义链主要是“KUBE-SERVICES”链、“KUBE-POSTROUTING”链、每个服务所对应的“KUBE-SVC-XXXXXXXXXXXXXXXX”链和“KUBE-SEP-XXXXXXXXXXXXXXXX”链），然后通过这些自定义链对流经到该node的数据包做DNAT和SNAT操作以实现路由、负载均衡和地址转换。

* ```链名的生成方式```

    先使用SHA256算法对"服务名+协议名+端口"生成哈希值，然后通过base32对该哈希值编码，最后取编码值的前16位

* ```iptables 管理周期```

    kubernetes调用iptables-save命令解析当前node中iptables的filter表和nat表中已经存在的chain，kubernetes会将这些chain存在两个map中（existingFilterChains和existingNATChains），然后再创建四个protobuf中的buffer（分别是filterChains、filterRules、natChains和natRules），后续kubernetes会往这四个buffer中写入大量iptables规则，最后再调用iptables-restore写回到当前node的iptables中。

    总的来说是: iptables-save 和 iptables-restore
