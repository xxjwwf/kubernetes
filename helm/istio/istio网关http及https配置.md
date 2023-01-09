正常的`http`配置
```YAML
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: httpbin-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80 # 默认是80端口
      name: http
      protocol: HTTP # 默认是http协议
    hosts:
    - "httpbin.example.com" # 网关域名入口，通常会解析到ingressgateway服务的ip上，然后前端再用nginx来做端口转发

---

apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - "httpbin.example.com"
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /status  # 路由匹配的前缀，必须以前缀开头，不然网关会报404
    - uri:
        prefix: /delay
    route:
    - destination:
        port:
          number: 8000   # 后端服务的端口，可以是nodeport，也可以是clusterip等
        host: httpbin    # 后端服务的名称缩写，全称应该是httpbin.<namespace>.svc.cluster.local.
```

正常的`https`配置，需要提前配置证书[官方文档](https://istio.io/latest/zh/docs/tasks/traffic-management/ingress/secure-ingress/)
```YAML
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: mygateway
spec:
  selector:
    istio: ingressgateway # use istio default ingress gateway
  servers:
  - port:
      number: 443        # 默认443端口
      name: https
      protocol: HTTPS    # 协议名称
    tls:
      mode: SIMPLE
      credentialName: httpbin-credential # 配置的证书secret资源名称
    hosts:
    - httpbin.example.com

---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - "httpbin.example.com"
    gateways:
  - mygateway
    http:
  - match:
    - uri:
        prefix: /status
    - uri:
        prefix: /delay
        route:
    - destination:
        port:
          number: 8000
        host: httpbin

```