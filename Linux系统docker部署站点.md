Linux系统docker部署站点步骤：

一、前期环境准备：

安装最新版docker：

1、更新yum版本：
 yum update

2、卸载docker旧版本(没有安装忽略此步骤)：
  yum remove docker docker-common docker-selinux docker-engine

3、安装需要的软件包：
  yum install -y yum-utils device-mapper-persistent-data lvm2

4、设置yum源：
  yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

5、查看有哪些可用的docker版本(可忽略步骤)：
  yum list docker-ce --showduplicates | sort -r

6、安装最新稳定版：
  yum install docker-ce -y 
  或者安装指定版本：
  yum -y install docker-ce-3:20.10.6-3.el7

7、启动docker：
  systemctl start docker

8、设置docker开机启动：
  systemctl enable docker.service

9、安装docker-compose：
  curl -L "https://github.com/docker/compose/releases/download/1.29.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

10、添加docker-compose权限：
  chmod +x /usr/local/bin/docker-compose

二、项目部署：

1、创建项目并创建Dockerfile文件，项目发布并将发布文件上传到linux系统

2、在上传发布文件的目录下执行打包命令：docker build -t 镜像名:版本 .

3、将docker-compose.yml文件上传到docker-compose安装目录，当前安装的目录地址为：
  /usr/local/bin/docker-compose (docker-compose.yml根据镜像包设置内容)

4、启动项目：
  docker-compose up -d
