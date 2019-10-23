# 记录Docker的一般性规则

包括命名、运行和仓库的一般规则

* 命名

    docker镜像的命名规则是: ```domain/namespace/image_name:tag```

    其中：domain指的是你的仓库的域名，k8s部署文件的namespace，官方的是从dockerhub上拉取的，```缺省为docker.io```。

    特别地，k8s.gcr.io=gcr.io/google_containers=gcr.io/google-containers, 所以gcr.io/google_containers/pause:3.0也可以用k8s.gcr.io/pause:3.0代替
