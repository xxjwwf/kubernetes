## 探针

**官网定义**

`
A probe is a diagnostic performed periodically by the kubelet on a container. To perform a diagnostic, the kubelet either executes code within the container, or makes a network request.
`

`
探针是一个在容器内运行的周期性的性能诊断工具。可以通过在容器内执行命令，或者是通过网络请求来完成对容器性能的诊断。
`

## 探针的使用方式

    1. exec（容器内执行命令）
    2. grpc（远程调用）
    3. httpGet（http的Get请求）
    4. tcpSocket（建立TCP socket连接）

**exec**

`
在容器内执行命令，如果命令执行状态码是0就表示健康，非0表示不健康
`


**grpc**

`
调用的status是SERVING的话表示健康，需要开通特性门，GRPCContainerProbe 
`

**httpGet**

`
通过请求容器的ip加端口，以及特定的url后缀，返回的http状态码在200-399之间表示健康
`

**tcpSocket**

`
尝试连接容器端口，如果能够连上表示健康，即使连上后被快速断开仍然算健康状态
`

## 探针的返回结果

    1. Success，表示探测成功
    2. Failure，表示探测失败
    3. Unknown，表示未知，后续不做处理，留给kubelet检查


## 探针的三种类型

 **livenessProbe**
 
 `
 存活性探针，如果探测失败kubelet会kill容器，并根据容器的重启策略进行操作
 `

**readinessProbe**
    
`
就绪性探针，诊断容器是否已经准备好对外提供服务。如果失败，endpoints控制器会将与该pod相关的所有服务从注册中心去除
`

**startupProbe**

`启动探针，诊断容器内的应用是否启动，如果提供了启动探测，则所有其他探测都将被禁用，直到它成功为止。如果失败，kubelet会kill容器，并根据容器的重启策略操作
`

## 探针应用场景

    1. 如果您希望您的容器在探测失败时被杀死并重新启动，则指定一个活跃restartPolicy度探测，并指定 Always 或 OnFailure

    2. 如果仅在探测成功时才开始向 Pod 发送流量，使用readinessProbe。在这种情况下，readiness probe 可能和 liveness probe 一样，
    但是readinessProbe会拒绝所有流量，直到探测成功。

    3. 如果应用对后端服务有严格的依赖，可以同时使用 livenessProbe 和 readinessProbe。当应用程序本身健康时，liveness 探针
    就会通过，但 readiness 探针还会检查每个所需的后端服务是否可用。这可以帮助避免将流量引导到只能以错误消息响应的 Pod

    4.容器需要在启动期间加载大数据、配置文件或迁移，您可以使用 启动探针


## 实践


[**livenessProbe-exec配置**](https://github.com/xxjwwf/kubernetes/blob/main/doc-yaml/web/%E6%8E%A2%E9%92%88/liveness-exec.yaml)



[**livenessProbe-httpGet配置**](https://github.com/xxjwwf/kubernetes/blob/main/doc-yaml/web/%E6%8E%A2%E9%92%88/liveness-httpGet.yaml)`使用httpGet的时候，如果请求的文件不在，就会出现诊断失败的情况，可以在pod启动后尝试修改web内相应文件名即可重现`


[**livenessProbe-tcpSocket配置**](https://github.com/xxjwwf/kubernetes/blob/main/doc-yaml/web/%E6%8E%A2%E9%92%88/liveness-tcpSocket.yaml)`当端口不存在时，就会诊断失败，但是容器还是running状态，只是会不断重启`


![存活探针](https://github.com/xxjwwf/kubernetes/blob/main/static/img/%E5%AD%98%E6%B4%BB%E6%8E%A2%E9%92%88.png)

`
存活探针在探测失败时，表现为容器的不断重启，直到达到上限。
`

**注意**

`
就绪探针和启动探针的配置基本一致，只是探针名称不同而已，因此不再重复配置实验
`
