# helm安装prometheus

`获取helm源和prometheus压缩包`
```bash
~]# helm repo add prometheus-community https://prometheus-community.github.io/helm-charts # 添加仓库
~]# helm pull prometheus-community/kube-prometheus-stack # 拉取文件
```

`prometheus operator部署`
```bash
上述步骤会拉取压缩包到本地
~]# tar -zxf kube-prometheus-stack-43.1.1.tgz
~]# cd kube-prometheus-stack
```
```bash
修改镜像仓库地址
~]# vim values.yaml
...
    patch:
      enabled: true
      image:
        #registry: k8s.gcr.io
        registry: docker.io
        #repository: ingress-nginx/kube-webhook-certgen
        repository: jettech/kube-webhook-certgen #拉取的是dockerhub的镜像
        tag: v1.3.0
        sha: ""
        pullPolicy: IfNotPresent
...
```

```bash
部署（可指定namespace）
~]# helm install prometheus ./
~]# kubectl get pods
~]# kubectl get svc
```
```bash
将服务类型修改为NodePort，主要用于本地访问测试（可选）
1.gragana要到子charts里面修改
2.其他服务（prometheus，alertmanager）都在当前values文件下改，prometheus是9090端口，alertmanager是9093
~]# helm ungrade prometheus ./ # 也可以kubectl edit直接修改
```

`服务路由配置`

>部署完isito后，可以将prometheus的各个服务组件暴露出来，使用istio的ingressgateway来转发

[prometheus路由配置](./yaml/prom-gw.yaml)</br>
[alertmanager路由配置](./yaml/alert-gw.yaml)</br>
[grafana路由配置](./yaml/grafana-gw.yaml)</br>


```bash
grafana默认密码：
admin/prom-operator # 可查看values文件里面的adminPassword字段
kubectl get secret -n default prometheus-grafana -o jsonpath="{.data.admin-user}" | base64 --decode ; echo
kubectl get secret -n default prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

`告警配置`

alertmanager-->中间件（dingtalk-webhook）-->钉钉

```bash
获取并配置中间件服务：
~]# git clone --depth=1 https://github.com/timonwong/prometheus-webhook-dingtalk.git
~]# cd prometheus-webhook-dingtalk/contrib/k8s/
需要修改两个文件：
1.kustomization.yaml # 注释以下三行，避免报错
#replicas:
#  - name: alertmanager-webhook-dingtalk
#    count: 1

2.config/config.yaml # 需要先在钉钉上建一个群，然后在群里配置机器人
targets:
  robot1: # 原本的字段名是webhook1，要改成robot1
    url: https://oapi.dingtalk.com/robot/send?access_token=#####
    secret: ####

启动中间件服务：
~]# cd prometheus-webhook-dingtalk/contrib/k8s
~]# kubectl apply -k .
~]# kubectl get svc,pod
```

`配置alertmanager`

```yaml
~]# cd kube-prometheus-stack
~]# vim values.yaml
...
  config:
    global:
      resolve_timeout: 5m
    inhibit_rules: # 告警抑制规则
      - source_matchers:
          - 'severity = critical'
        target_matchers:
          - 'severity =~ warning|info'
        equal:
          - 'namespace'
          - 'alertname'
      - source_matchers:
          - 'severity = warning'
        target_matchers:
          - 'severity = info'
        equal:
          - 'namespace'
          - 'alertname'
      - source_matchers:
          - 'alertname = InfoInhibitor'
        target_matchers:
          - 'severity = info'
        equal:
          - 'namespace'
    route:
      group_by: ['namespace'] # 告警分组规则
      group_wait: 30s 
      group_interval: 5m
      repeat_interval: 12h # 告警重复发生的间隔
      receiver: 'webhook' # 默认接收人
      routes:
      - receiver: 'webhook' # 定义一个接收路由规则，接收人是webhook，可定义多个
        matchers:
          - alertname =~ "InfoInhibitor|Watchdog"
    receivers:              # 定义接收人，可定义多个
    - name: 'webhook'
      webhook_configs:      # 接收类型为webhoook，可以是钉钉，微信，短信，电话等接口
      - url: http://alertmanager-webhook-dingtalk/dingtalk/robot1/send
        send_resolved: true  # 告警恢复时也发送
    templates:
    - '/etc/alertmanager/config/*.tmpl' # alertmanager内告警模板的存放目录

告警模板配置，注意空格缩进
...

  templateFiles:
  ## An example template:，
    template_1.tmpl: |-
           {{ define "cluster" }}{{ .ExternalURL | reReplaceAll ".*alertmanager\\.(.*)" "$1" }}{{ end }}

           {{ define "slack.myorg.text" }}
           {{- $root := . -}}
           {{ range .Alerts }}
             *Alert:* {{ .Annotations.summary }} - `{{ .Labels.severity }}`
             *Cluster:* {{ template "cluster" $root }}
             *Description:* {{ .Annotations.description }}
             *Graph:* <{{ .GeneratorURL }}|:chart_with_upwards_trend:>
             *Runbook:* <{{ .Annotations.runbook }}|:spiral_note_pad:>
             *Details:*
               {{ range .Labels.SortedPairs }} - *{{ .Name }}:* `{{ .Value }}`
               {{ end }}
           {{ end }}
           {{ end }}
...

使用helm更新服务
~]# cd kube-prometheus-stack
~]# helm upgrade prometheus .
```

1.钉钉发送服务的config.yaml文件配置问题webhook
2.模板问题
3.钉钉关键字问题

`servicemonitor模板`  [帮助文档](https://help.aliyun.com/document_detail/260895.html)

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: es-md-monitor # svcmonitor的名称
  namespace: default # svcmonitor所在的名称空间
spec:
  endpoints:
    - interval: 15s # 拉取指标的间隔
      path: /metrics # 拉取指标的默认后缀
      port: http  # 后端服务的端口名字
  namespaceSelector:  # 指定svcmonitor拉取哪些命名空间下的服务指标
    any: true         # 表示为所有名称空间
  selector:           # 匹配服务
    matchLabels:      # 匹配以下两个标签即可
      app: prometheus-elasticsearch-exporter
      release: zcy-elasticsearch
```
> kubectl apply -f svcmonitor.yaml
> 登陆prometheus后台，查看configuration内容

`查看指标列表`
```bash
先根据prometheus中的target来定位指标的路径
~]# kubectl run alpine --image=alpine:3.12 --command sleep 36000
~]# kubectl exec -it alpine -- apk --update add curl
~]# kubectl exec -it alpine -- sh
/]# curl podIP:port/suffix
```

`常用命令`

```bash
~]# kubectl get prometheusrule # 获取告警规则
~]# kubectl get alertmanager,prometheus # 查看组件版本
```