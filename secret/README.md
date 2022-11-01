1.使用yaml创建，先要登录harbor，然后cat /root/.docker/config.json | base64 -w 0(去掉换行符)
最后使用yaml创建
2.使用命令创建，`kubectl create secret docker-registry harbor-secretname242 --docker-server=chinapopin.com:18443 --docker-username=username --docker-password=passwd -n test-project`
