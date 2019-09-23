# 主要记录在k8s编写和调试crontab任务的事项

在Linux编写定时任务一般是使用```crontab```。那么在k8s里面执行一个定时任务呢？其实原理是一样的，不过有一些细小的差别。所以，首先需要了解一下如何编写crontab。

## Linux环境

* 基本格式

    ```*　　*　　*　　*　　*```　　command
    ```分　 时　 日　 月　 周```　 命令

* 解释和示例

    1. 第1列表示分钟：00~59(每分钟可以用/1表示)

    2. 第2列表示小时：00~23(特别地, 0表示0点)

    3. 第3列表示日：00~31

    4. 第4列表示月份：00~12

    5. 第5列表示星期：0~6(0表示星期天)

    6. 第6列表示：需要执行的命令

    示例：
        在每天0点执行时间同步：```00 00 * * * ntpdate -u time.pool.aliyun.com```

## k8s

在了解了crontab后，我们就可以在k8s里面尝试编写一个定时任务[Cronjob](https://kubernetes.io/zh/docs/tasks/job/automated-tasks-with-cron-jobs/)了，这里以定时清理文件夹为例。

* 我首先编写了一个遍历某个文件夹下面的所有文件或文件夹，以清理特定的文件或文件夹

* 编写Cronjob

    ```yaml
    apiVersion: batch/v1beta1
    kind: CronJob
    metadata:
    name: clean
    namespace: clean
    spec:
    schedule: "00 00 * * *" # 分 时 日 月 周 命令
    jobTemplate:
        spec:
        template:
            spec:
            containers:
            - name: clean
                imagePullPolicy: Always
                image: clean
                command: ["/bin/bash", '-c', "python3 /apps/clean.py"]
                env:
                - name: style-migration
                value: /apps/static
                volumeMounts:
                - name: imgsave
                mountPath: /apps/static
            restartPolicy: OnFailure
            volumes:
            - name: imgsave
                persistentVolumeClaim:
                claimName: style-migration-pvc
    ```
