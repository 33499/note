>docker安装Nginx

```shell
# 1.搜索镜像 seach
# 2.下载镜像 pull
# 3.运行测试
docker images
# -d 后台运行
# --name 给容器起名字
# -p 宿主机端口：容器内部端口
docker run -d --name nginx01 -p 3344:80 nginx
curl localhost:3344
# 4.进入容器
docker exec -it nginx01 /bin/bash
whereis nginx
```

>docker安装tomcat

```shell
#官方
docker run -it --rm tomcat:9.0 #用完即删

#下载并启动
docker pull tomcat:9.0
docker images
docker run -d -p 3355:8080 --name tomcat01 tomcat
#进入容器
docker exec -it tomcat01 /bin/bash

```



>docker 容器于与主机传递数据

```shell
docker cp 本机文件路径 容器id:/root/
# 将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中、
docker cp  96f7f14e99ab:/www /tmp/
```

>netstat命令的安装

```shell
apt -y install net-tools
netstat -ano | grep "80" @查看80端口是否被占用
```

>docker安装ubuntu配置nginx

```shell
docker run -itd --name ubt -p 80:80 -p 1935:1935 ubuntu /bin/bash
docker exec -it ubt /bin/bash
apt install nginx

```

>docker centos安装

```shell
1．卸载旧版本：
    yum remove docker \
              docker-client \
              docker-client-latest \
              docker-common \
              docker-latest \
              docker-latest-logrotate \
              docker-logrotate \
              docker-engine
2.需要的安装包
    yum install -y yum-utils
3. 设置镜像仓库
    yum-config-manager \
           --add-repo \
           https://download.docker.com/linux/centos/docker-ce.repo  默认国外的（不建议）
    yum-config-manager \
           --add-repo \
            http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo      阿里云镜像地址（建议使用）
    更新 yum 缓存：
        yum makecache fast
4. 安装docker引擎
    yum install docker-ce docker-ce-cli containerd.io
5. 启动Docker。
    systemctl start docker
6. 查看Docker。
    docker version
7. 测试helloword
    docker run hello-world
8. 查看镜像
    docker images
```

>docker ubuntu安装

```shell
#1．卸载旧版本：
    sudo apt-get remove docker docker-engine docker.io containerd runc
#2.更新apt软件包索引并安装软件包以允许apt通过HTTPS使用存储库
    sudo apt-get update
    sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg-agent \
     software-properties-common
#3.添加Docker的官方GPG密钥
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
 //通过搜索指纹的后8个字符，验证现在是否拥有带有指纹的密钥 
    sudo apt-key fingerprint 0EBFCD88
 //结果：
 pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
 uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
 sub   rsa4096 2017-02-22 [S]

#4. 添加仓库
   sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

#5.安装docer引擎
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io
#6. 启动并查看Docker
    service docker start
    docker version
#7. 测试helloword
    sudo docker run hello-world
8. 查看镜像
    docker images
```

