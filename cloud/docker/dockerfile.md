# [Dokcerfile 指南](https://docs.docker.com/engine/reference/builder/)

## Usage

docker build根据Dockerfile和context构建镜像。其中context 表示当前的上下文内容，</br>表示指定的路径或URL下的一系列文件。
官方文档里面说明```A context is processed recursively```，</br>所以PATH包含其中的子文件夹，URL包含```Git repository and its submodules```</br>假定你是在/home/yourproject这个目录下执行build操作，则当前的上下文内容就是该路径下的所有文件和文件夹。</br>

```bash
# 一般用法
$ docker build -f /path/to/a/Dockerfile .

# 同时标注多个tag, 多个-t 参数
$ docker build -t shykes/myapp:1.0.2 -t shykes/myapp:latest .

# 多阶段构建，是否使用cache，--cache-from缓存来源等更多用法请参考文档
```

```The build is run by the Docker daemon, not by the CLI```所以在构建镜像时会递归地发送context到daemon，</br>
所以最佳的做法是尽量在一个空文件放置Dockerfile和必要的文件，或者像.gitignore一样的做法编写一个.dockerignore文件

构建好镜像后，应该考虑如何将镜像上传到仓库: ```docker push username/repo:tag```
