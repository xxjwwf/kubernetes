1.使用yaml创建，先要登录harbor，然后cat /root/.docker/config.json | base64 -w 0(去掉换行符)
最后使用yaml创建

2.使用命令创建，`kubectl create secret docker-registry harbor-secretname242 --docker-server=harbor.xiejiawen.cn --docker-username=username --docker-password=passwd -n test-project`

    docker-registry：指定secret名称
    --docker-server: 指定harbor地址
    --docker-username: 指定登录harbor的用户名，建议使用机器人账号
    --docker-password: 登录密码
    -n: 指定namespace

