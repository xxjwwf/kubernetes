1.创建harbor机器人账号成功，且主机成功登录harbor，使用/root/.docker/config.json里的auth创建secret的时候失败
```bash 
Secret in version "v1" cannot be handled as a Secret
```
**解决：** base64编码不对，可能是前后空格对不上，或者是最后一行有空格


2.解决第一个问题后，再次运行`kubectl apply -f` 出错
```bash
The Secret "harbor-robot-secret" is invalid: data[.dockerconfigjson]: Invalid value: "<secret contents redacted>": invalid character 'r' looking for beginning of value
```
---
**解决：** 
base64还是有问题，得使用不换行模式进行转码：`cat ~/.docker/config.json |base64 -w 0`

---

3.nfs读写权限挂载到pod上后，pod不能读写，宿主机也不能读写此目录
  决：nfs需要对相应目录的rwx权限也要设置，如果要赋写权限，那么目录必须要有w权限

---

4.我的harbor网站是放在自己的电脑虚拟机里面的，每次不用的时候就将虚拟机挂起，但是我发现每次挂起前服务都是能从电脑本机正常访问的，挂起完再恢复虚拟机时，从本机电脑上就不能访问了。

**排查：**
	
  - a.从虚拟机本机访问是正常的，从电脑端问harbor不正常，由此可判断是网络问题，但是每次重启虚拟机之后，harbor就正常访问了
  - b.由此可判断出某些默认设置可能在挂起虚拟机的时候发生了改变，经goole发现是有一个内核参数需要改,修改后问题仍然存在，发现是容器网络出现问题，每次得删除容器重建才行，目前还未想到更好的办法
  ```bash
  docker-compose down // 停止并删除容器，网络，卷和映像。
  docker-compose up -d //启动服务 -d 后台模式
  ```

**解决：**
```bash 
  echo net.ipv4.ip_forward=1  >> /etc/sysctl.conf
  sysctl -p
  systemctl restart network.service
```

5.不同节点之间的pod无法通讯


**现象**

  - 在每个节点上，pod可以互相访问节点内的pod，并且pod可以ping通节点的物理网卡地址
  - 节点之间可以相互ping通，节点间防火墙规则也已开放
  - nodePort模式下，比如某个服务只在A节点有部署，那么就只能通过A节点的ip:nodeport来访问，通过其他节点的ip:nodeport访问是不通的（按道理应该是可以通过其他节点ip:nodeport来访问服务的）
  - A节点不能ping通B节点的Pod的ip

**解决过程**

  - 重启flannel插件，无效
  - 删除flannel插件，重装calico插件（部分成功，master02仍然不能通过nodeport正常访问相关服务）
  - 经检查后，发现master02是新加的，ntpdate没有安装导致时间不一致，因此不能正常访问
  - 安装ntpdate后，并同步时间重启服务器后正常


**成功后的截图如下：**

![master1](https://github.com/xxjwwf/kubernetes/blob/main/static/img/master01-calico.png)


![master2](https://github.com/xxjwwf/kubernetes/blob/main/static/img/master02-calico.png)


`
在每个master上都能看到其他master和其他节点的网络配置，相当于是个路由表
`