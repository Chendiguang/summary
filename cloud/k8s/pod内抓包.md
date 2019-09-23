# 在pod的容器内抓包

* 在master上获取pod id

    ```bash
    function get_id() {
        set -eu
        ns=${2-"default"}
        pod=`kubectl -n $ns describe pod $1 | grep -A10 "^Containers:" | grep -Eo 'docker://.*$' | head -n 1 | sed 's/docker:\/\/\(.*\)$/\1/'`
        echo $pod
    }

    # get_id pod_name namespace
    ```

* 在工作节点上进入容器内部，利用宿主机抓包

    ```bash
    function cmd() {
        pid=`docker inspect -f {{.State.Pid}} $1`
        cmd="nsenter -n --target $pid"
        echo $cmd
        $cmd
    }

    # cmd pod_id
    # tcpdump -i eth0 -w test.pcap port 80
    ```
