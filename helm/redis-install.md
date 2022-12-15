helm安装redis后，没有pv，新建一个pv后还是无法匹配

> helm repo add bitnami/redis
> 
> helm install redis-test bitnami/redis 

```bash
# 安装完redis后，查看redis pod
~]# kubectl get pods -n default
~]# kubectl get pods  
NAME                     READY   STATUS             RESTARTS   AGE
redis-test-master-0      0/1     Pending            0          11m
redis-test-replicas-0    0/1     Pending            0          11m
```
`查看pod的内部报错`
```bash
~]# kubectl describe pods redis-test-master-0
  ...
  Warning  FailedScheduling  31s   default-scheduler  0/5 nodes are available: 5 pod has unbound immediate PersistentVolumeClaims.
  Warning  FailedScheduling  31s   default-scheduler  0/5 nodes are available: 5 pod has unbound immediate PersistentVolumeClaims.
```
`查看pvc的状态，发现是pending状态`
```bash
~]# kubectl get pvc
NAME                    STATUS    VOLUME                CAPACITY   ACCESS MODES   STORAGECLASS          AGE
redis-volume-master-2   Pending    ...                                            
```
 

`根据以上信息发现应该是pvc没有绑定`

`1.所以我就先弄个nfs并建一个pvc试试。`

```bash
~]# yum install nfs-utils
~]# echo '/data2 *(rw) >> /etc/exports'
~]# echo '/data3 *(rw) >> /etc/exports'
~]# systemctl restart nfs-server && systemctl restart rpcbind
```
`重新创建pv和pvc，这里我只创建了redis-master的pvc和pv，replicas的pv和pvc语法是一样的，只需目录和名字区分开即可`
```yaml
# pv-master
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-volume-master
spec:
  capacity:
    storage: 2Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: redis-volume-master
  nfs:
    path: /data2
    server: 10.110.101.111

--- # pvc-master
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: redis-volume-master
spec:
  storageClassName: redis-volume-master # 这个字段的值要跟pv一致
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```
`查看一下，发现pvc的状态已经是Bound状态了`
```bash
~]# kubectl get pvc # 上面我省略了redis-volume-slave的配置
NAME                  STATUS   VOLUME                CAPACITY   ACCESS MODES   STORAGECLASS          AGE
redis-volume-master   Bound    redis-volume-master   2Gi        RWO            redis-volume-master   127m
redis-volume-slave    Bound    redis-volume-slave    2Gi        RWO            redis-volume-slave    86m
```
`2.pvc绑定好后，需要再配置到redis的values文件里面去，values文件里面可能有多个persistence，只需要关注enabled为true的区域即可，bitnami/redis里面是master和replicas各有一个persistence`
```bash
~]# grep -v '^#' values.yaml | grep -v "^[[:space:]]\+#" | grep -v '^$'  > values-bak.yaml # 导出有用的配置
~]# grep  persistence values-bak.yaml # 查看有几处需要配置pvc
```
`values.yaml文件里面，persistence的上下文一般是这样：`
```bash
...
  persistence:
    enabled: true                       # true表示启用这个配置
    medium: ""
    sizeLimit: ""
    path: /data
    subPath: ""
    subPathExpr: ""
    storageClass: ""
    accessModes:
      - ReadWriteOnce
    size: 8Gi                           # 可以注释，我们已经手动创建了pvc
    annotations: {}
    selector: {}
    dataSource: {}
    existingClaim: redis-volume-master # 填刚刚创建好的pvc名称，字段不能重复，会被覆盖
...
```
`将刚刚helm安装的reids卸载掉`
```bash
~]# helm uninstall redis-test
```
`重新安装redis，并且指定刚刚修改的那个为values文件`
```bash
~]# helm install redis-test2 bitnami/redis -f values-bak.yaml

# 安装好后，再看看pod的状态，发现已经是running了
~]# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
redis-test2-master-0     1/1     Running   0          2m22s
```
`根据templates/NOTES.txt或者是刚刚的输出结果，可以尝试连接一下redis`
```bash
# 1.获取redis密码
~]# kubectl get secret --namespace default redis-test3 -o jsonpath="{.data.redis-password}" | base64 -d

# 2.kubectl连接到redis的pod里面
~]# kubectl exec --tty -i redis-test2-master-0 --namespace default -- bash

# 3.pod里面连接redis
~]# redis-cli -h 127.0.0.1 -p 6379
127.0.0.1:6379> auth KHU9wJc2iy # 使用刚刚获取到的密码
OK
```

> 至此使用helm安装redis完毕，解决了pvc的绑定问题


