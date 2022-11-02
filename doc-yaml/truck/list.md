1.创建harbor机器人账号成功，且主机成功登录harbor，使用/root/.docker/config.json里的auth创建secret的时候失败
报
```bash 
Secret in version "v1" cannot be handled as a Secret
```
解决：base64编码不对，可能是前后空格对不上，或者是最后一行有空格

2.解决第一个问题后，再次运行kubectl apply -f 出错
报```bash
The Secret "harbor-robot-secret" is invalid: data[.dockerconfigjson]: Invalid value: "<secret contents redacted>": invalid character 'r' looking for beginning of value
```
解决：base64还是有问题，得使用不换行模式进行转码：`cat ~/.docker/config.json |base64 -w 0`

3.nfs读写权限挂载到pod上后，pod不能读写，宿主机也不能读写此目录
解决：nfs需要对相应目录的rwx权限也要设置，如果要赋写权限，那么目录必须要有w权限
