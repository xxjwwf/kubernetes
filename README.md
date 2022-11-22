
学习kubernetes
----
### [搭建harbor](https://github.com/xxjwwf/kubernetes/tree/main/harbor)
用于本地测试快速拉取镜像

### [创建harbor的secret](https://github.com/xxjwwf/kubernetes/tree/main/doc-yaml/secret)
用于登录harbor拉取镜像

### [安装nfs](https://github.com/xxjwwf/kubernetes/tree/main/nfs)

nfs主要用于k8s内部的存储挂载，可用与web项目


### [web项目](https://github.com/xxjwwf/kubernetes/tree/main/doc-yaml/web)

完成tomcat镜像的构建，资源挂载nfs等，并完成一系列的测试

---
### 控制器

#### [ingress-nginx]()


DemonSet
每个节点运行gcluster、ceph存储
每个节点运行logstash、fluentd
每个节点运行监控、collected、Datadog

Job、CronJob用于批处理脚本

StatefulSet

HPA(Horizontal Pod Autoscaling)