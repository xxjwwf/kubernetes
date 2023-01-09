### Istio安装
---

#### 背景
> 1.istio属于集群级资源，因此正常情况下一个集群安装一个就够了，安装多个会报错</BR>
> 2.ingressgateway最重要的就是gateway和virtualservice这两个配置</BR>
> 3.gateway主要就是用于http协议，可以是http也可以是https</BR>
> 4.virtualservice的路由配置很重要，这是流量分发的基础