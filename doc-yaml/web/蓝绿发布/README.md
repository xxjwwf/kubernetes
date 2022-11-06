## 蓝绿发布


`蓝绿发布是一种零宕机的应用更新策略。进行蓝绿发布时，应用的旧版本服务与新版本服务会同时并存，并且新版本和旧版本共用一个服务入口，更新的时候，只需要将上层的服务入口切换为新版本即可，一旦发现新版本出现异常，可以通过同样的版本切换方式快速切回老版本，如果新服务无异常，则可删除老版本的配置即可。
`

**优点**
```
- 快速切换，即使服务出现不可用也能快速回滚，风险较小
- 发布过程简单，前期配置好后只需切换服务版本即可
```
**缺点**


```
- 资源占用较高，需要同等复制全部资源，成本较高
```

### 配置

**[beta环境配置](https://github.com/xxjwwf/kubernetes/blob/main/doc-yaml/web/%E8%93%9D%E7%BB%BF%E5%8F%91%E5%B8%83/tomcat-beta.yaml)**

**[prod环境配置](https://github.com/xxjwwf/kubernetes/blob/main/doc-yaml/web/%E8%93%9D%E7%BB%BF%E5%8F%91%E5%B8%83/tomcat-prod.yaml)**

**[服务配置](https://github.com/xxjwwf/kubernetes/blob/main/doc-yaml/web/%E8%93%9D%E7%BB%BF%E5%8F%91%E5%B8%83/tomcat-svc.yaml)**

### 蓝绿切换
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-tomcat
  labels:
    app: web
    type: backend
spec:
  type: NodePort
  ports:
  - port: 8080
    nodePort: 30012
  selector:
    version: beta  # 每次修改selector中的标签即可
```