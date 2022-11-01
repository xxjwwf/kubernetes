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
