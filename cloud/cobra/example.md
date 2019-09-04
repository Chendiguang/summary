# Cobra 入门范例

Cobra作为一个在云原生应用中被广泛使用的命令行库，它提供了简单的接口来创建命令行程序，所以有必要对其的用法进行简单的了解。

## 安装Cobra

```bash
go get -u github.com/spf13/cobra/cobra
# 网络不好时，先下载到GOPATH下对应目录，然后解决依赖，再build
git clone https://github.com/spf13/cobra.git
```

## 使用

这跟kubectl 里面的效果是一样的，可以对比理解。因为k8s里面的命令行实现就是基于Cobra

```bash
# 初始化一个项目
cobra init myapp
# go build main.go -o myapp
./myapp

# 给myapp添加一个子命令，第一级子命令错误时，还能给出提示
cobra add version

./myapp version

# 多级子命令
cobra add get
cobra add pods -p getCmd

./myapp get pods
```
