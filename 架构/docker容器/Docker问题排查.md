# Docker问题排查

1. 查看容器死亡原因： `docker inspect [CONTAINER] -f '{{json .State}}'` 

    ```json
    ## 输入如下
    {
        "Status": "exited",
        "Running": false,
        "Paused": false,
        "Restarting": false,
        "OOMKilled": true,
        "Dead": false,
        "Pid": 0,
        "ExitCode": 137,
        "Error": "",
        "StartedAt": "2019-12-03T02:02:27.564618662Z",
        "FinishedAt": "2019-12-03T02:49:14.023816567Z"
    }
    ```

2. `docker stats [CONTAINER]`  监控容器资源消耗

3. `docker logs jd-service | grep -i MaxHeapSize` 启动项加`-XX:+PrintFlagsFinal -XX:+PrintGCDetails ` 查看启动之后堆信息

4. oom 之后  输出dump信息 `-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/logs/`

5. jdk8_131之后才支持 jdk10之后不支持 jvm不识别容器与宿主机的区别问题修改 `-XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap` 

