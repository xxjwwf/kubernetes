## nfs的安装及注意事项

### 注意点
1.nfs是网络文件系统，所以网络上一定要对客户端放行<br />
2.nfs是一个共享文件系统，所以分配权限的时候要谨慎，能不给写权限就不要给<br />
3.客户端用户能压缩成普通用户就压缩成普通用户，安全需要<br />
4.nfs是通过/etc/exports文件来映射目录及赋权的，保护好这个文件<br />
5.nfs使用读权限将目录挂载到客户端后，即使这个目录权限是有写权限的，客户端也是readOnly状态<br />
6.nfs使用读写权限将目录挂载到客户端后，如果目录没有相应的写权限，那么客户端也是readOnly状态<br />
7.综上所述，nfs的权限是要取文件夹权限和配置权限的交集，缺一不可<br />

---
### 安装过程
**1.配置镜像源**
```bash
cat>/tmp/epel.repo<<EOF
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel-7&arch=$basearch&infra=$infra&content=$contentdir
failovermethod=priority
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
EOF
```

**2.安装nfs-utils及rpcbind**

```bash
yum install nfs-utils 及rpcbind
```


**3.添加配置** # 可以配置多行
```bash
echo '/data 192.168.99.0/24(ro)' >> /etc/exports
```
**启动**
```bash
systemctl start nfs && systemctl enabl nfs
systemctl start rpcbind && systemctl enable rpcbind
```

