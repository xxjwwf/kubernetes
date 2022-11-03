
学习kubernetes
----
### 1.搭建harbor
**注意点**

harbor最好是内网搭建，便于快速拉取镜像
```bash
#离线安装模式下安装，官方harbor下载链接[https://github.com/goharbor/harbor/releases],可以选择合适的版本进行下载。

1.解压安装包:bash $ tar xzvf harbor-offline-installer-version.tgz

2.修改harbor.yaml文件，并且配置证书，建议使用个人阿里云账号申请免费证书，每年都有20个免费额度，腾讯云也有类似功能

3.运行Install.sh脚本，会自动安装并且通过docker-compose启动harbor，启动后就可以使用https访问了。

```

### 安装nfs ###

nfs主要用于k8s内部的存储挂载，可用与web项目
