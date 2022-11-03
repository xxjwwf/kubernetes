
### tomcat服务的基本实现：
  1.实现tomcat日志目录的外部外载，可实时查看日志
  2.实现webapps目录的外部挂载，可实时更新web内容,实现蓝绿部署
  3.实现tomcat服务的入口配置，ingress
  4.实现rolebinding及serviceAccountName，实现configmap

---
## 进阶一 tomcat挂载目录变更为nfs，更便于管理
  1.搭建nfs服务
  2.根据初始化目录，可根据服务类型简单分类为web或者data
  3.将nfs挂载到tomcat上，实现集中挂载管理

---
## 进阶二 日志收集系统
  1. 搭建日志收集服务
  2. 从nfs中，或者直接从服务pod中获取日志
  3. 可视化管理
 

