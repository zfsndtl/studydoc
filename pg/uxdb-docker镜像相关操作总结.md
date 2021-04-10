# uxdb-docker镜像相关操作总结

## 1.docker环境搭建

**1.下载Dcoker依的赖环境**

1. 想安装Docker，需要先将依赖的环境全部下载下来，就像Maven依赖JDK一样
2. yum -y     install yum-utils device-mapper-persistent-data lvm2

**2.指定Docker镜像源**

1. 默认下载Docker会去国外服务器下载，速度较慢，可以设置为阿里云镜像源，速度更快
2. yum-config-manager     --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

**3.安装Docker**

1. yum makecache fast
2. yum -y     install docker-ce
3. 查看版本 docker version

![image-20201022112030218](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022112030218.png)

**4.启动Docker并测试**

1. 安装成功后，需要手动启动，设置为开机启动，并测试一下     Docker
2. \#启动docker服务
3. systemctl     start docker
4. \#设置开机自动启动
5. systemctl     enable docker
6. \#测试
7. docker     run hello-world

## 2.uxdb镜像安装

**1.获取镜像文件** 

[ftp://192.29.1.2/build/uxdbbuild/uxdb-docker/uxdb-server-linux7-2.1.0.2-SE-docker.tar.gz](ftp://192.29.1.2/build/uxdbbuild/uxdb-docker/uxdb-server-linux7-2.1.0.2-SE-docker.tar.gz)

**2.解压缩**

 tar xvf uxdb-server-linux7-2.1.0.2-SE-docker.tar.gz

生成uxdb-server-linux7-2.1.0.2-SE-docker.tar文件

**3.导入镜像**

 docker load < uxdb-server-linux7-2.1.0.2-SE-docker.tar

**4.查看镜像是否导入成功** 

docker image ls

**5.生成容器并运行** 

 docker container run -it --network host uxdb-server-linux72.1.0.2-se /bin/bash

 docker run -itd --name uxdb0.0.1 -p 5432:5432  192.29.1.2:8999/library/uxdb-server-linux7-2.1.0.2-ee

**6..进入容器、使用** 

进入容器后默认ROOT用户，uxdb用户已经创建，密码123456





## 3.docker仓库

**1.首先搭建一个docker私有库服务**

docker run -d -p 5000:5000 --restart=always --name registry2 registry:2

**2.给要上传的服务打tag**

由于docker默认镜像仓库是dockerhub，所以java:my相当于docker.io/java:my，因此，想要将镜像推送到私服仓库中，需要修改镜像标签。

docker tag  192.29.1.2:8999/library/uxdb-server-linux7-2.1.0.2-ee 192.73.0.254:5000/uxdb0.0.1

![image-20201022095950937](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022095950937.png)

![image-20201022103647693](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022103647693.png)

**3.上传镜像**

docker push 192.73.0.254:5000/uxdb0.0.1

![image-20201022095919919](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022095919919.png)

对私有仓库的操作，其提供了HTTP API 地址为:https://docs.docker.com/registry/spec/api/

![image-20201022104136583](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022104136583.png)

![image-20201022104207759](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022104207759.png)

4.查看私服镜像所有仓库**

curl http://192.73.0.254:5000/v2/_catalog

![image-20201022100216520](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022100216520.png)

**5.查看仓库中镜像的所有标签列表**

curl  http://192.73.0.254:5000/v2/uxdb0.0.1/tags/list

 ![image-20201022100556351](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022100556351.png)

```shell
curl --get  http://192.73.0.254:5000/v2/uxdb0.0.1/tags/list

{"name":"uxdb0.0.1","tags":["latest"]}
```



**6.查看仓库中的image详情**

GET /v2/<name>/manifests/<reference>

![image-20201022104031609](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022104031609.png)

<!--The `name` and `reference` parameter identify the image and are required. The reference may include a tag or digest.-->

curl -get  http://192.73.0.254:5000/v2/uxdb0.0.1/mainfests/latest



上述操作，需要首先获取摘要digest

curl --header "Accept: application/vnd.docker.distribution.manifest.v2+json"  -X HEAD  http://192.73.0.254:5000/v2/uxdb0.0.1/mainfests/latest

**7.删除仓库中的镜像**

![image-20201022104002967](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022104002967.png)

![image-20201022104856767](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022104856767.png)

由此可以看出API需要一个name参数和一个摘要参数，下面提示说如果是2.3及以后的服务，HEAD 必须包含下面的内容才能获取到正确的摘要,我们先来获取这个摘要

**8.加入密码**

构建仓库用的密码文件：username 123abc是用户名密码

docker run --entrypoint htpasswd registry -Bbn username 123abc >> /registry-var/auth/htpasswd）

下面是完整的加上自定义配置文件、存储目录的命令了：

docker run -d -p 5000:5000 --restart=always -v /registry-var/auth/:/auth/ -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd -v /registry-var/my_registry:/var/lib/registry/ -v /registry-var/config.yml:/etc/docker/registry/config.yml --name registry registry

docker run -d -p 5000:5000 --restart=always -v /registry-var/auth/:/auth/ -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" -e "REGISTRY_STORAGE_DELETE_ENABLED=true" -v /registry-var/my_registry:/var/lib/registry/ --name registry registry

登陆测试：

docker login -u 用户名-p 密码 registry.x.com:5000

出来Login Succeeded

## 4.uxdb镜像使用

**1.查看镜像**

docker images

![image-20201022164820492](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022164820492.png)

**2.启动镜像**

 docker run -itd --name uxdb0.0.1 -p 5432:5432  192.29.1.2:8999/library/uxdb-server-linux7-2.1.0.2-ee



```
仓库中下载使用镜像
vi  /etc/docker/daemon.json 
{"insecure-registries":["192.73.0.254"]}

docker login 192.30.0.247
用户名：admin
密码：Harbor12345

systemctl daemon-reload 

systemctl restart docker

docker pull 192.30.0.247/uxdb/uxdb-docker:2.1.1.3.01

docker volume create uxdb2.1.1.3.01

docker run -itd --name uxdb2.1.1.3.01 -p 1234:1234 -v uxdb2.1.1.3.01:/hoem/uxdb/uxdbinstall  192.30.0.247/uxdb/uxdb-docker:2.1.1.3.01

docker run -itd --name uxdb2.1.1.3.01 -p 1234:1234 -v uxdb2.1.1.3.01:/home/uxdb/uxdbinstall  192.73.0.254/test/uxdb-docker:2.1.1.3.01
```

![image-20201109202828735](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201109202828735.png)



**3.查看容器状态**

docker ps

![image-20201022164925773](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022164925773.png)

**4.进入容器**

![image-20201022165111749](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022165111749.png)

**5.退出容器**

![image-20201022165349376](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022165349376.png)

**6.停止容器**

docker stop 容器name or id

![image-20201022165530753](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022165530753.png)

## 5.docker file打包uxdb镜像



```dockerfile
#dockerfile 在/home/uxdb/uxdbinstall 目录下

FROM centos

MAINTAINER dufz



#切换镜像目录（类似cd命令），进入/usr目录
WORKDIR /usr

#在/usr/下创建jdk目录,用来存放jdk文件
RUN mkdir jdk1.8

#将宿主机的jdk目录下的文件拷至镜像的/usr/jdk目录下
COPY jdk1.8.0_131 /usr/jdk1.8

RUN useradd uxdb
#RUN groupadd uxdb
RUN usermod -a -G uxdb uxdb

WORKDIR /home/uxdb

RUN mkdir -p uxdbinstall/local_uxfs

#将宿主机的uxdbinstall目录下的文件拷至镜像的/home/uxdb/uxdbinstall目录下

COPY local_uxfs /home/uxdb/uxdbinstall/local_uxfs

RUN echo " export LANG=zh_CN.UTF-8" >> /home/uxdb/.bashrc
RUN echo " export LC_ALL=zh_CN.UTF-8" >> /home/uxdb/.bashrc

#可以使用ll命令
RUN echo "alias ll='ls $LS_OPTIONS -l'" >> /home/uxdb/.bashrc
RUN echo "alias ll='ls $LS_OPTIONS -l'" >> /root/.bashrc
#uxdb目录权限设置，默认是root:root
RUN chown -R uxdb:uxdb /home/uxdb

#RUN export LANG=zh_CN.UTF-8
#RUN export LC_ALL=zh_CN.UTF-8
#RUN export LANG=zh_CN.UTF-8


#设置环境变量
ENV JAVA_HOME=/usr/jdk1.8
ENV JRE_HOME=$JAVA_HOME/jre
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib:$CLASSPATH
ENV PATH=/sbin:$JAVA_HOME/bin:$PATH
#公开端口
#EXPOSE 8080
RUN echo $JAVA_HOME
RUN echo over
USER uxdb
```

```shell
#设置不需要打包到镜像中的文件目录

touch .dockerignore 
vi .dockerignore 

dbsql
deploy
encryptLicense
license
thirdparty
uxdbagent
uxfs
uxpool
```

执行打包脚本

```shell
 docker  build -t dockerfiletest:0.0.1 .
```

查看打包的镜像，运行容器查看jdk及字符集

```shell
#运行镜像
docker run --name dockerfiletest -p 5455:5455 -itd dockerfiletest:0.0.1

#查看容器
docker ps

#进入容器
docker exec -it 98 "/bin/bash"
```

![image-20201026143512381](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026143512381.png)

```shell
#生产环境dockerfile -247服务器

# .dockerignore 文件
cat .dockerignore 
auto-install.xml
encryptLicense  
installer  
uninstaller  
uxfs
.dockerignore
Dockerfile

#Dockerfile文件
FROM centos:7
MAINTAINER uxsino


RUN useradd uxdb
#RUN groupadd uxdb
RUN usermod -a -G uxdb uxdb

WORKDIR /home/uxdb

RUN mkdir  uxdbinstall

#将宿主机的uxdbinstall目录下的文件拷至镜像的/home/uxdb/uxdbinstall目录下

COPY ./ /home/uxdb/uxdbinstall/

RUN echo " export LANG=zh_CN.UTF-8" >> /home/uxdb/.bashrc
#可以使用ll命令
RUN echo "alias ll='ls $LS_OPTIONS -l'" >> /home/uxdb/.bashrc
RUN echo "alias ll='ls $LS_OPTIONS -l'" >> /root/.bashrc
RUN chown -R uxdb:uxdb /home/uxdb

RUN echo over
USER uxdb


#运行镜像
docker run --name dockerfiletest -p 5455:5455 -itd dockerfiletest:0.0.1

#查看容器
docker ps

#进入容器
docker exec -it 98 "/bin/bash"

```

![image-20201026164932116](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026164932116.png)

![image-20201026175052061](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026175052061.png)

![image-20201026175106589](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026175106589.png)

## 6.卷使用相关

**1.查看卷**

![image-20201022165617830](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022165617830.png)

**2.创建卷**

sudo docker volume create uxdb_volume

![image-20201022165817464](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022165817464.png)

**3.删除卷**

sudo docker volume rm uxdb_volume

![image-20201022165820645](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022165820645.png)

**4.启动容器并挂载/home/uxdb/uxdbinstall/dbsql目录到卷目录**

sudo docker run -it --name uxdbtest-volume  -p 5433:5433  -v uxdb_volume:/home/uxdb/uxdbinstall/dbsql

d 192.73.0.254:5000/uxdb0.0.1

**5.查看容器挂载信息**

docker inspect 2cc

![image-20201026180059064](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026180059064.png)

![image-20201026180134957](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026180134957.png)

![image-20201026180332729](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026180332729.png)

## 7.**docker-compose安装与使用**-Harbor

```
安装docker-compose
两种最新的docker安装方式
1.root用户从github上下载docker-compose二进制文件安装
下载最新版的docker-compose文件 
 sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

添加可执行权限  
sudo chmod +x /usr/local/bin/docker-compose
 
测试安装结果  

$ docker-compose --version

 

docker-compose version 1.3.3, build 1719ceb
2.pip安装
安装python-pip
yum -y install epel-release
yum -y install python-pip
sudo pip install docker-compose

  查看版本：
docker-compose --version

2、docker-compose常用命令

docker-compose -h                           # 查看帮助

docker-compose up                           # 创建并运行所有容器
docker-compose up -d                        # 创建并后台运行所有容器
docker-compose -f docker-compose.yml up -d  # 指定模板
docker-compose down                         # 停止并删除容器、网络、卷、镜像。

docker-compose logs       # 查看容器输出日志
docker-compose pull       # 拉取依赖镜像
dokcer-compose config     # 检查配置
dokcer-compose config -q  # 检查配置，有问题才有输出

docker-compose restart   # 重启服务
docker-compose start     # 启动服务
docker-compose stop      # 停止服务
```

![image-20201027094227385](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027094227385.png)

## 8.docker **离线安装Harbor**

```shell
1.下载版本，比如2.1.0 https://github.com/goharbor/harbor/releases 
2.上传并解压 tar -zxvf harbor.v2.1.0.tar.gz到目录/home/uxdb/harbor
cd /home/uxdb/harbor
vi harbor.yml
./prepare
./install.sh
```

[[docker harbor使用参考文档 ](https://www.cnblogs.com/snail-gao/p/12046044.html)]

关闭https协议，使用http协议访问（HTTP protocol is insecure. Harbor will deprecate http protocol in the future. Please make sure to upgrade to https HTTP协议不安全。 Harbor将来会弃用http协议。 请确保升级到https）

![image-20201027102021905](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027102021905.png)

./prepare

![image-20201026184137576](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026184137576.png)

![image-20201027100337623](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027100337623.png)

./install.sh

![image-20201027100749155](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027100749155.png)

![image-20201027100815139](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027100815139.png)

![image-20201027100856862](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027100856862.png)

![image-20201027101353037](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027101353037.png)

![image-20201027101638266](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027101638266.png)

![image-20201027101722783](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027101722783.png)

http://192.73.0.254/harbor/sign-in?redirect_url=%2Fharbor%2Fprojects

默认用户名为:admin,密码为:Harbor12345

![image-20201027124035541](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027124035541.png)

![image-20201027124250402](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027124250402.png)

上传镜像失败，必须是https的

![image-20201027145621735](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027145621735.png)

修改docker为 http请求---非http得的在服务器192.30.0.247上实验的

```shell


需要登陆
docker login 192.30.0.247
vi  /etc/docker/daemon.json 
{"insecure-registries":["192.30.0.247"]}

systemctl daemon-reload 

systemctl restart docker

systemctl status docker.service  查看服务状态

docker tag hello-world:latest 192.30.0.247/test/hello-world:0.1
docker push 192.30.0.247/test/hello-world:0.1

#需要登陆
docker login 192.30.0.247

Username: admin
Password: Harbor12345

```

![image-20201103173954082](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103173954082.png)

![image-20201103173851294](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103173851294.png)

![image-20201103173909775](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103173909775.png)



**https**

```shell

[root@monitor harbor]# mkdir /data/cert -p
[root@monitor harbor]# cd /data/cert
[root@monitor cert]# openssl genrsa -out /data/cert/ca_harbor.key 2048
Generating RSA private key, 2048 bit long modulus
..+++
......................................+++
e is 65537 (0x10001)
[root@monitor cert]# openssl req -x509 -new -nodes -key /data/cert/ca_harbor.key -subj "/CN=www.dfz.com" -days 5000 -out /data/cert/ca_harbor.crt

#-----------------------来自官网-------------------------------
#https://goharbor.io/docs/2.0.0/install-config/configure-https/
#1.Generate a Certificate Authority Certificate
#1.1Generate a CA certificate private key.
openssl genrsa -out /data/cert/ca.key 4096
#1.2Generate the CA certificate.
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=192.73.0.254" \
 -key /data/cert/ca.key \
 -out /data/cert/ca.crt
 
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=www.dfz.com" \
 -key /data/cert/ca.key \
 -out /data/cert/ca.crt 
 
#2.Generate a Server Certificate 
#2.1.Generate a private key.
openssl genrsa -out /data/cert/192.73.0.254.key 4096
openssl genrsa -out /data/cert/www.dfz.com.key 4096

#2.2.Generate a certificate signing request (CSR). 
openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=192.73.0.254" \
    -key /data/cert/192.73.0.254.key \
    -out /data/cert/192.73.0.254.csr
    
openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=www.dfz.com" \
    -key /data/cert/www.dfz.com.key \
    -out /data/cert/www.dfz.com.csr    
#2.3.Generate an x509 v3 extension file.
#第一种方式：域名
cat > /data/cert/v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1=www.dfz.com
EOF    

openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=www.dfz.com" \
    -key /data/cert/www.dfz.com.key \
    -out /data/cert/www.dfz.com.csr 
第二种方式ip
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = IP:192.73.0.254
EOF

#2.4.Use the v3.ext file to generate a certificate for your Harbor host.
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in 192.73.0.254.csr \
    -out 192.73.0.254.crt

openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in www.dfz.com.csr \
    -out www.dfz.com.crt



#3. 修改harbor.yml
vi harbor.yml

https:
  # https port for harbor, default is 443
  port: 443
  # The path of cert and key files for nginx
  certificate: /data/cert/192.73.0.254.crt
  private_key: /data/cert/192.73.0.254.key


#4.After generating the ca.crt, yourdomain.com.crt, and yourdomain.com.key files, you must provide them to Harbor and to Docker, and reconfigure Harbor to use them.

#4.1 Copy the server certificate and key into the certficates folder on your Harbor host.
cp 192.73.0.254.crt /data/cert/
cp 192.73.0.254.key /data/cert/

cp www.dfz.com.crt /data/cert/
cp www.dfz.com.key /data/cert/

#4.2 Convert yourdomain.com.crt to yourdomain.com.cert, for use by Docker.
openssl x509 -inform PEM -in 192.73.0.254.crt -out 192.73.0.254.cert

# 4.3 Copy the server certificate, key and CA files into the Docker certificates folder on the Harbor host. You must create the appropriate folders first.
mkdir -p /etc/docker/certs.d/192.73.0.254/
cp 192.73.0.254.cert /etc/docker/certs.d/192.73.0.254/
cp 192.73.0.254.key /etc/docker/certs.d/192.73.0.254/
cp ca.crt /etc/docker/certs.d/192.73.0.254/

mkdir -p /etc/docker/certs.d/www.dfz.com/
cp www.dfz.com.cert /etc/docker/certs.d/www.dfz.com/
cp www.dfz.com.key /etc/docker/certs.d/www.dfz.com/
cp ca.crt /etc/docker/certs.d/www.dfz.com/

#5 Restart Docker Engine.
systemctl restart docker

docker start $(docker ps -aq)

 #-----------------------来自官网-------------------------------

docker rmi --force $(docker images | grep goharbor | awk '{print $3}') 
#批量删除镜像  --harbor镜像 name都是以goharbor开头

# 重新生成配置文件
./prepare --with-notary --with-clair --with-chartmuseum
# ./prepare 
docker-compose down
./install.sh



#上传镜像
#1. 制造tag
docker tag hello-world:latest  192.73.0.254/test/hello-world:0.0.1

#2.push
docker push 192.73.0.254/test/hello-world:0.0.1

#3.uxdb镜像上传
[root@www harbor]# docker tag 192.73.0.254:5000/uxdb0.0.1:latest 192.73.0.254/test/uxdb:0.0.1
[root@www harbor]# docker push 192.73.0.254/test/uxdb

```

**1.IP方式**

**1.1.修改配置及生成https验证**

![image-20201103102039725](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103102039725.png)

![image-20201103104800707](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103104800707.png)



![image-20201103110151017](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103110151017.png)

**1.2.启动**

![image-20201103110113859](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103110113859.png)

**1.3登录，创建项目后上传镜像**

![image-20201103110257719](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103110257719.png)

![image-20201103110450304](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103110450304.png)



![image-20201103110747751](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103110747751.png)

![image-20201103111013024](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103111013024.png)

![image-20201103111040105](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103111040105.png)

![image-20201103111844029](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201103111844029.png)



**2.域名方式**

![image-20201102171352629](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201102171352629.png)











![image-20201027151718601](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027151718601.png)

docker tag  hello-world  www.dfz.com/hello-world





当前问题阶段

![image-20201027160959108](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027160959108.png)

## 10.docker环境搭建过程中遇到的问题

**1.http: server gave HTTP response to HTTPS client仓库上传镜像时报错**

**解决方案:**

出现这问题的原因是：Docker自从1.3.X之后docker registry交互默认使用的是HTTPS，但是搭建私有镜像默认使用的是HTTP服务，所以与私有镜像交时出现以上错误。

这个报错是在本地上传私有镜像的时候遇到的报错：

![img](https://images2017.cnblogs.com/blog/1205699/201712/1205699-20171206005341972-493924765.png)

解决办法是：在docker server启动的时候，增加启动参数，默认使用HTTP访问：

 vim /usr/lib/systemd/system/docker.service

![img](https://images2017.cnblogs.com/blog/1205699/201712/1205699-20171206005808566-147554548.png)

在12行后面增加 --insecure-registry 192.73.0.254:5000

或者：

docker login 192.30.0.247
vi  /etc/docker/daemon.json 
{"insecure-registries":["192.73.0.254"]}

修改好后重启docker 服务

systemctl daemon-reload 

systemctl restart docker

systemctl status docker.service  查看服务状态

![img](https://images2017.cnblogs.com/blog/1205699/201712/1205699-20171206005949269-388068348.png)

重启docker服务后，将容器重启

docker start $(docker ps -aq)

![image-20201022102905489](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201022102905489.png)



[问题参考]: https://www.cnblogs.com/programmer-tlh/p/10996443.html

**2.Dockerfile Build镜像过大或者一直在传送内容**

当使用Dockerfile Build镜像时，

现象1. 有时会发现发送到Daemo的内容过大，如下：

Sending build context to Docker daemon 218.2 MB

现象2. 并导致生成的docker image过大

而，Dockerfile中的内容却不多，

FROM ceph-client
 MAINTAINER dev <dev@xtaotech.com>

RUN yum clean all && yum makecache && yum install -y metaview-server metaview-cli

ADD entrypoint.sh /entrypoint.sh

WORKDIR /
 ENTRYPOINT ["/entrypoint.sh"]

百度后发现，docker client会默认把Dockerfile同级所有文件发给docker Deamon中，因为目录下有备份的tar文件，有几百兆

解决办法有两种：

1.使用.dockerignore文件，设置黑名单，该文件包含的目录不会被发送到Docker daemon中

2.将Dockerfile迁移后其他目录中执行。

3.将不需要的文件删除

![image-20201026112250170](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026112250170.png)

![image-20201026112600001](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026112600001.png)

![image-20201026112656719](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026112656719.png)

![image-20201026113218098](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026113218098.png)

**3.dockerfile 制作镜像使用sudo命令时出错**

![image-20201026114746654](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026114746654.png)

**4.解决Docker ADD/COPY 报ADD failed: stat /var/lib/docker/tmp/docker-builder: no such file or director**

![image-20201026140423545](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026140423545.png)

意思就是说。ADD [source] [target] 命令找不到source的文件

搜了大致有以下情况：

没有source文件，

或者source文件跟Dockerfile不在同一目录，

或者命令docker build -t 传参数传错了，命令里跟add后面文件名字不一样。

![image-20201026140503436](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201026140503436.png)

注意：此处的原因是因为 COPY source的相对路径写错，不能写成../uxdbinstall/local_uxfs或者uxdbinstall/local_uxfs，因为dockerfile文件就在uxdbinstall目录，要选择相对路径

5.**uxdb类库相关**

基准镜像 centos:7

**6.docker仓库搭建好后，外部机器Telnet ip:port ，及浏览器无法访问。本机crul命令正常**

原因：**虚拟机使用docker 外部机器无法访问端口问题**

使用虚拟机启动docker镜像之后，外部宿主机无法访问指定端口服务

宿主机是a ,虚拟机是b 。虚拟机没有可视化界面，在b上启动docker服务后发现A不能访问

1，排查防火墙firewall-cmd --state

如果输出的是“not running”则FirewallD没有在运行，且所有的防护策略都没有启动，那么可以排除防火墙阻断连接的情况了。

如果输出的是“running”，表示当前FirewallD正在运行，则关闭防火墙

二、ip转发没有打开

```
执行 sysctl net.ipv4.ip_forward
```

显示net.ipv4.ip_forward=0则表示未打开。

harbor.yml 

如果上述文件中的值为0,说明禁止进行IP转发；如果是1,则说明IP转发功能已经打开,

要想打开IP转发功能，可以直接修改上述文件：echo 1 > /proc/sys/net/ipv4/ip_forward

把文件的内容由0修改为1。禁用IP转发则把1改为0。

上面的命令并没有保存对IP转发配置的更改，下次系统启动时仍会使用原来的值，要想永久修改IP转发，需要修改

/etc/sysctl.conf文件，修改下面一行的值：

net.ipv4.ip_forward = 1

修改后可以重启系统来使修改生效，也可以执行下面的命令来使修改生效：

sysctl -p /etc/sysctl.conf

进行了上面的配置后，IP转发功能就永久开启了。

```shell
[root@monitor harbor]# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
[root@monitor harbor]# echo 1 > /proc/sys/net/ipv4/ip_forward
[root@monitor harbor]# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1

```

![image-20201027114936176](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201027114936176.png)

[虚拟机使用docker 外部机器无法访问端口问题](https://blog.csdn.net/xiaosannimei/article/details/104570016)



6.[部署harbor仓库相关问题总结 ](https://www.cnblogs.com/kvipwang/p/9896712.html)

下面的问题是因为，https协议生成的crt证书名，key值 要与hostname对应

![image-20201102192031657](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201102192031657.png)

![image-20201030180219021](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201030180219021.png)

![image-20201102163035672](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201102163035672.png)

上传镜像异常，猜测试ssl验证失败了，其次注意一定要生成 2.4步骤中的 192.73.0.254.crt文件，harbor.yml配置文件也配置证书必须是crt文件格式，否则nginx模块验证会失败，导致此服务起不来，后续一直是restart,也无法通过浏览器访问









切换到harbor目录
docker-compose down 停止所有服务

docker-compose up -d 启动所用服务

卸载：停止服务后 
使用：docker rmi --force $(docker images | grep goharbor | awk '{print $3}') 批量删除镜像  --harbor镜像 name都是以goharbor开头

![image-20201102163014957](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201102163014957.png)

注：harbor.cfg文件修改后 使用 ./install.sh 启动前 先删掉 /data/secretkey  


如果重新部署 遇到：vmware/harbor-db 状态非up 删除 /data/database目录 重新启动即可。

登录前先修改：

daemon.json 文件

{
  "insecure-registries" : ["IP"]
}

**7.中文环境问题**

```
FROM centos 
MAINTAINER maocheng li 
#设置系统编码 
RUN yum install kde-l10n-Chinese -y 
RUN yum install glibc-common -y 
RUN localedef -c -f UTF-8 -i zh_CN zh_CN.utf8 
#RUN export LANG=zh_CN.UTF-8 
#RUN echo "export LANG=zh_CN.UTF-8" >> /etc/locale.conf 
#ENV LANG zh_CN.UTF-8 
ENV LC_ALL zh_CN.UTF-8
```



## [11.免sudo使用docker命令](https://www.cnblogs.com/mafeng/p/8683914.html)             

**背景**   

因为使用的是sudo安装docker，所以会导致一个问题。以普通用户登录的状况下，在使用`docker images`时必须添加`sudo`，那么如何让docker免`sudo`依然可用呢？于是开始搜索解决方案。

**查看docker权限**

ll /run|grep docker

属于docker用户组；讲当前用户添加到docker组



![image-20201106164126176](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201106164126176.png)

**添加用户到docker组**

sudo gpasswd -a ${USER} docker
#正在将用户“uxdb”加入到“docker”组中

#sudo gpasswd -d ${USER} docker

#正在将用户“uxdb”从“docker”组中删除

**使当前会话窗口组设置生效，或者重启会话**
uxdb@uxdb:~$ newgrp - docker

**重启docker服务**

uxdb@uxdb:~$ sudo service docker restart

## 12 docker  file传入参数

### 语法

```
ARG <name>[=<default value>]
```

### 作用 和 描述

ARG 指令使用 --build-arg = 标志定义一个变量，用户可以使用 docker build 命令在构建时将该变量传递给构建器。如果用户指定了未在 Dockerfile 中定义的构建参数，则构建会输出告警。

```
[Warning] One or more build-args [foo] were not consumed.
```

Dockerfile 可以包含一个或多个 ARG 指令。例如，以下是有效的 Dockerfile:

```
FROM busybox
ARG user1
ARG buildno
1234
```

警告:
建议不要使用构建时变量来传递 github 密钥，用户凭证等秘密。使用 docker history 命令可以使镜像的任何用户都可以看到构建时变量值。

### 默认值(Default values)

ARG 指令可以选择包含默认值:

```
FROM busybox
ARG user1=someuser
ARG buildno=1
1234
```

如果 ARG 指令具有默认值，并且在构建时没有传递值，则构建器将使用默认值。

## Scope

ARG 变量定义从 Dockerfile 中定义的行开始生效，而不是从命令行或其它地方的参数使用。例如，考虑这个 Dockerfile:

```
1 FROM busybox
2 USER ${user:-some_user}
3 ARG user
4 USER $user
...

```

用户通过调用以下内容构建此文件：

```
$ docker build --build-arg user=what_user .
```

第 2 行的 USER 评估为 some_user，因为在后续第 3 行定义了用户变量。第 4 行的 USER 在定义用户时评估为 what_user，并在命令行上传递 what_user 值。在通过 ARG 指令定义之前，对变量的任何使用都会导致空字符串。
ARG 指令在构建阶段结束时超出范围。要在多个阶段中使用 arg，每个阶段必须包含 ARG 指令。

```
FROM busybox
ARG SETTINGS
RUN ./run/setup $SETTINGS

FROM busybox
ARG SETTINGS
RUN ./run/other $SETTINGS
1234567
```

### 使用 ARG 变量(Using ARG variables)

你可以使用 ARG 或 ENV 指令指定 RUN 指令可用的变量。使用 ENV 指令定义的环境变量始终覆盖同名的 ARG 指令。考虑这个带有 ENV 和 ARG 指令的 Dockerfile。

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER v1.0.0
4 RUN echo $CONT_IMG_VER
1234
```

然后，假设使用此命令构建此镜像:

```
$ docker build --build-arg CONT_IMG_VER=v2.0.1 .
1
```

在这种情况下，RUN 指令使用 v1.0.0 而不是用户传递的 ARG 设置:v2.0.1此行为类似于 shell 脚本，其中本地作用域的变量会覆盖作为参数传递或从环境继承的变量，定一点。

使用上面的示例，但不同的 ENV 规范，你可以在 ARG 和 ENV 指令之间创建更有用的交互,例如将 ARG 值传递给 ENV 使用：

```
1 FROM ubuntu
2 ARG CONT_IMG_VER
3 ENV CONT_IMG_VER ${CONT_IMG_VER:-v1.0.0}
4 RUN echo $CONT_IMG_VER
```

与 ARG 指令不同，ENV 值始终保留在构建的镜像中。考虑没有 --build-arg 标志的 docker 构建:

```
$ docker build .
```

使用此 Dockerfile 示例，CONT_IMG_VER 仍然保留在镜像中，但其值为 v1.0.0，因为它是 ENV 指令在第 3 行中的默认设置。
本例中的变量扩展技术允许你从命令行传递参数，并通过利用 ENV 指令将它们保存在最终镜像中。只有一组有限的 Dockerfile 指令支持变量扩展。

### 预定义的 ARG(Predefined ARGs)

Docker 有一组预定义的 ARG 变量，你可以在 Dockerfile 中使用而无需相应的 ARG 指令。

- HTTP_PROXY
- http_proxy
- HTTPS_PROXY
- https_proxy
- FTP_PROXY
- ftp_proxy
- NO_PROXY
- no_proxy

要使用它们，只需使用标志在命令行上传递它们：

```
--build-arg <varname>=<value>
```

默认情况下，这些预定义变量将从 docker history 的输出中排除。排除它们可以降低在 HTTP_PROXY 变量中意外泄漏敏感验证信息的风险。

例如，考虑使用 --build-arg 构建以下 Dockerfile HTTP_PROXY=http://user:pass@proxy.lon.example.com

```
FROM ubuntu
RUN echo "Hello World"
```

在这种情况下，HTTP_PROXY 变量的值在 docker 历史记录中不可用，并且不会被缓存。如果要更改位置，并且代理服务器已更改为http://user:pass@proxy.sfo.example.com，则后续构建不会导致缓存未命中。

如果你需要覆盖此行为，则可以通过在 Dockerfile 中添加 ARG 语句来执行此操作，如下所示:

```
FROM ubuntu
ARG HTTP_PROXY
RUN echo "Hello World"
```

构建此 Dockerfile 时，HTTP_PROXY 将保留在 docker 历史记录中，并且更改其值会使构建缓存无效。
@TODO



### 方案1：dockerfile :uxdb dockerfile build时创建默认实例

```dockerfile
vi Dockerfile

FROM 192.71.1.11/test/uxdb:0.0.1

MAINTAINER dufz

#docker file build时实例参数配置,主要用于创建images时创建实例
#docker file build时实例参数配置，如果不设置就用默认的，否则用配置的
#但是构建时设置的环境变量，对于启动CMD来讲无法获取
ARG INSTANCE_NAME=test
ARG UXDB_PASSWORD=1
ARG UXDB_USERNAME=uxdb

RUN echo $UXDB_PASSWORD $INSTANCE_NAME

#切换镜像目录（类似cd命令），进入uxdb安装目录
WORKDIR /home/uxdb/uxdbinstall/dbsql/bin
USER uxdb
RUN mkdir /home/uxdb/uxdbinstall/dbsql/data
RUN echo $UXDB_PASSWORD >> /home/uxdb/uxdbinstall/dbsql/data/password.txt

#设置环境变量，只用于运行时创建实例
ENV PATH=/home/uxdb/uxdbinstall/dbsql/bin:$PATH
ENV UXDATA=/home/uxdb/uxdbinstall/dbsql/data

#ENV INSTANCE_NAME=test
#ENV UXDB_PASSWORD=1
#RUN echo $UXDB_PASSWORD

#RUN uxdb -V
RUN cat /home/uxdb/uxdbinstall/dbsql/data/password.txt
RUN initdb -D $INSTANCE_NAME --pwfile /home/uxdb/uxdbinstall/dbsql/data/password.txt
RUN echo over
EXPOSE 5432
#CMD ["/bin/bash","-c","ux_ctl test start"]

```

```
创建uxdb镜像包含 uxdbtest的默认实例，密码为1

docker  build -t  dockerfiletest:0.0.1 --build-arg INSTANCE_NAME="uxdbtest"  --build-arg UXDB_PASSWORD=1 .

```

### ARG与ENV区别

格式：`ARG <参数名>[=<默认值>]`

构建参数和 `ENV` 的效果一样，都是设置环境变量。所不同的是，`ARG` 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 `ARG` 保存密码之类的信息，因为 `docker history` 还是可以看到所有值的。

`Dockerfile` 中的 `ARG` 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 `docker build` 中用 `--build-arg <参数名>=<值>` 来覆盖。

在 1.13 之前的版本，要求 `--build-arg` 中的参数名，必须在 `Dockerfile` 中用 `ARG` 定义过了，换句话说，就是 `--build-arg` 指定的参数，必须在 `Dockerfile` 中使用了。如果对应参数没有被使用，则会报错退出构建。从 1.13 开始，这种严格的限制被放开，不再报错退出，而是显示警告信息，并继续构建。这对于使用 CI 系统，用同样的构建流程构建不同的 `Dockerfile` 的时候比较有帮助，避免构建命令必须根据每个 Dockerfile 的内容修改。



### 方案2：dockerfile 在容器运行时创建实例

由于build时的实例设置参数，在cmd运行时无法获得，Dockerfile文件修改如下：

```dockerfile
cat Dockerfile
FROM 192.71.1.11/test/uxdb:0.0.1

MAINTAINER dufz



#切换镜像目录（类似cd命令），进入uxdb安装目录
WORKDIR /home/uxdb/uxdbinstall/dbsql/bin
USER uxdb
COPY ./uxdb.lic /home/uxdb/uxdbinstall/license
#设置环境变量，只要用于运行时创建实例
ENV PATH=/home/uxdb/uxdbinstall/dbsql/bin:$PATH
ENV UXDATA=/home/uxdb/uxdbinstall/dbsql/data

ENV INSTANCE_NAME=test
ENV UXDB_PASSWORD=1

RUN echo $UXDB_PASSWORD $INSTANCE_NAME
RUN mkdir /home/uxdb/uxdbinstall/dbsql/data
#RUN echo $UXDB_PASSWORD >> /home/uxdb/uxdbinstall/dbsql/data/password.txt

#RUN uxdb -V
#RUN cat /home/uxdb/uxdbinstall/dbsql/data/password.txt
#RUN initdb -D $INSTANCE_NAME --pwfile /home/uxdb/uxdbinstall/dbsql/data/password.txt
RUN echo over
EXPOSE 5432
CMD ["/bin/bash","-c","echo $UXDB_PASSWORD >> /home/uxdb/uxdbinstall/dbsql/data/password.txt&&\
initdb -D $INSTANCE_NAME --pwfile /home/uxdb/uxdbinstall/dbsql/data/password.txt\
&&uxdb -D $INSTANCE_NAME&&rm -rf /home/uxdb/uxdbinstall/dbsql/data/password.txt"]


```

```
创建uxdb镜像包含 uxdbtest的默认实例，密码为1

docker  build -t  dockerfiletest:0.0.1

docker run -it --name dockertest -p 5432:5432 -e INSTANCE_NAME="uxdbtest" -e UXDB_PASSWORD=1 -d dockerfiletest:0.0.1



```

但是如果将实例的创建放在CMD中，会导致 容器创建成功后，docker start 容器都会执行CMD脚本，导致实例重复创建报错，docker logs查看容器启动日志

![image-20210105111119692](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20210105111119692.png)

方案3：build时创建默认实例，cmd只负责启动实例，注意实例名称的传递就不能写在arg参数中

```dockerfile
cat Dockerfile
FROM 192.71.1.11/test/uxdb:0.0.1

MAINTAINER dufz



#切换镜像目录（类似cd命令），进入uxdb安装目录
WORKDIR /home/uxdb/uxdbinstall/dbsql/bin
USER uxdb
COPY ./uxdb.lic /home/uxdb/uxdbinstall/license
#设置环境变量，只要用于运行时创建实例
ENV PATH=/home/uxdb/uxdbinstall/dbsql/bin:$PATH
ENV UXDATA=/home/uxdb/uxdbinstall/dbsql/data

ENV INSTANCE_NAME=test
ENV UXDB_PASSWORD=1

RUN echo $UXDB_PASSWORD $INSTANCE_NAME
RUN mkdir /home/uxdb/uxdbinstall/dbsql/data
RUN echo $UXDB_PASSWORD >> /home/uxdb/uxdbinstall/dbsql/data/password.txt

#RUN uxdb -V
RUN cat /home/uxdb/uxdbinstall/dbsql/data/password.txt
RUN initdb -D $INSTANCE_NAME --pwfile /home/uxdb/uxdbinstall/dbsql/data/password.txt
RUN rm -rf  /home/uxdb/uxdbinstall/dbsql/data/password.txt
RUN echo initdb -D $INSTANCE_NAME success
EXPOSE 5432
CMD ["/bin/bash","-c","uxdb -D test"]



```





jenkensDockerBuildWithDefaultInstance.sh 

```shell
docker run -it --name 192.71.1.11/uxdb/uxdb-docker -p 5432:5432  -e UXDB_PASSWORD=1 -d 192.71.1.11/uxdb/uxdb-docker:2.1.1.3.02




```

