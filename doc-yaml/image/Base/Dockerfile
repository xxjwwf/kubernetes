FROM centos:7
MAINTAINER 谢佳文<xxjwwf@163.com>
ADD jdk-19_linux-x64_bin.tar.gz /usr/local/
ADD apache-tomcat-9.0.68.tar.gz /usr/local/
RUN yum -y install vim
#设置环境
ENV MYPATH /usr/local
#设置工作目录
WORKDIR $MYPATH
#设置JAVA_HOME 环境变量，后边版本对应上方安装包
ENV JAVA_HOME /usr/local/jdk-19.0.1
#设置CLASSPATH 环境变量
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
#设置CATALINA_HOME 环境变量
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.68
#设置CATALINA_BASE 环境变量
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.68
#设置PATH
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#暴露端口
EXPOSE 8080
#指定容器启动后执行的命令
CMD bash /usr/local/apache-tomcat-9.0.68/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.68/logs/catalina.out
