# 关于k8s部署文件的一些总结

    主要总结在编写yaml时的一注意事项。

* ```编写在一个deploy.yaml 还是按功能分别编写多个文件```

    一般而言，只用同一个的部署文件相对容易管理，更改的同一个配置简单。不过总的来说差别不大，根据个人喜好或公司规范来实施。

* ```namespace```

    最好在每个功能的metadata处都标明namespace，方便资源管理。例如之前碰到一个删除deployment和pod不成功的示例，根据kubelet日志排查的时候发现是由于etcd的数据出错导致的，最终需要删除namespace才能删除。

* ```访问集群内部的服务```

    url: ```http://svc-name.namesapce-name:port/../..``` 大概是这种格式，后面部分按照你的ingress配置进行修改。

* ```健康检查```

  健康检查是比较有必要的，可以及时报告现在的网络连通性

```yaml
  readinessProbe:
    httpGet:s
      path: /healthz
      port: 11011
    scheme: HTTP
  initialDelaySeconds: 5
  periodSeconds: 2
  timeoutSeconds: 1
  successThreshold: 1
  failureThreshold: 5
```
