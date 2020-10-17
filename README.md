# 应用docker部署Concerto
Concerto是一个开源的在线自适应测验平台，由剑桥大学心理测量中心开发。用户可以通过其内置的模板构建自适应性测验。本文主要介绍如何在VPS上应用docker部署Concerto平台。包括三个步骤：
1. 开通VPS服务
2. 在VPS上安装docker
3. 在docker上部署Concerto
## 1. 开通VPS服务
选择一个VPS服务商，购买VPS服务器。如国内的阿里云、腾讯；国外的谷歌云、亚马逊等。网上有很多搭建VPS的教程可以参考，这里就不再赘述。

## 2. 在VPS上安装docker
### docker的概念
docker是一种容器解决方案，由一系列集成的技术构成，主要为开发、分享和运行基于容器的应用程序提供解决方案。首先，docker作为一种桌面和开发者工具，对于新的容器化应用程序，可以提供流线型的交付过程，对于现有应用程序，可以提供简化的容器方案，帮助其未来的完善。其次，docker拥有世界上最大的容器镜像库，为跨团队、跨平台的协作与分享提供便利。最后，docker可以通过容器运行容器化应用程序。

使用Linux容器部署应用程序称为容器化。容器并不是一个新的概念，但是将它用于轻松部署应用程序确实一个全新的尝试。
容器化越来越受欢迎，因为容器是：
- 灵活：即使是最复杂的应用也可以容器化。
- 轻量级：容器利用并共享主机内核。
- 可互换：可以即时部署更新和升级。
- 便携式：可以在本地构建，部署到云服务器，并在任何地方运行。
- 可扩展：可以增加并自动分发容器副本。
- 可堆叠：可以垂直和即时堆叠服务。

不同的VPS服务器系统，其安装过程存在差异，本文以Centos为例，介绍如何应用docker的镜像仓库安装docker。

### 设置仓库
1. 安装所需的程序包。`yum-utils`提供`yum-config-manager`的功能，`devicemapper`存储驱动需要`device-mapper-persistent-data`和`lvm2`。
```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
2. 应用如下的命令设置稳定的仓库。
```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
### 安装docker引擎
1. 安装最新版本的docker引擎
```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```
2. 启动docker
```
$ sudo systemctl start docker
```
3. 通过运行`hello-world`镜像来验证是否成功安装docker引擎。
```
$ sudo docker run hello-world
```
此命令会下载测试镜像并在容器中运行它，当容器成功运行时，它会反馈一条信息并退出。

直到此，已经在VPS成功安装docker服务。
## 3. 在docker上部署Conterto
Compose是一个用于定义和运行多容器Docker应用程序的工具。安装Compose之后，可以使用YAML文件来配置多个应用程序的服务。这样，使用单个命令，就能够配置中创建并启动所有服务。

使用Compose包括三个步骤：

1. 应用`Dockerfile`定义您的应用程序环境，以便可以在任何地方进行复制。

2. 应用`docker-compose.yml`定义构成应用程序的服务，以便它们可以在隔离的环境中一起运行。

3. 通过`Run docker-compose up`在Compose启动并运行整个应用程序。
安装Portainer

### 安装docker-compose

1. 运行此命令以下载Docker-Compose的当前稳定版本
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
2. 赋予二进制文件可执行权限：
```
sudo chmod +x /usr/local/bin/docker-compose
```
### 利用docker-compose创建Conterto

1. 首先登录[concerto-platform](https://github.com/campsych/concerto-platform/tree/c7424b4a4d0d17b3a4a447f18b0afcdcbdc96ef4)

2. 然后下载该文件压缩包。

3. 解压缩包，然后进入文件夹:`usr/concerto/doker-compose`

4. 将该文件夹中的文件`docker-compose.yml`更改为：
```
version: '3.0'
services:
    database:
      image: mysql:5.7
      container_name: database
      restart: unless-stopped
      ports:
        - "3306:3306"
      environment:
        - MYSQL_ROOT_PASSWORD=root
        - MYSQL_DATABASE=concerto
        - TZ=Europe/London
      volumes:
        - ./database/mysql:/var/lib/mysql
        - ./database/conf/mysqld.cnf:/etc/mysql/conf.d/mysqld.cnf
        - ./logs/database:/var/log/mysql

    concerto:
      build: ./concerto
      image: yaozeyang90/concerto-platform:5.0.beta.7.4
      container_name: concerto
      restart: unless-stopped
      volumes:
        - ./concerto/config/parameters.yml:/usr/src/concerto/app/config/parameters.yml
        - ./concerto/config/parameters_administration.yml:/usr/src/concerto/app/config/parameters_administration.yml
        - ./concerto/config/parameters_test_runner.yml:/usr/src/concerto/app/config/parameters_test_runner.yml
        - ./concerto/config/parameters_uio.yml:/usr/src/concerto/app/config/parameters_uio.yml
        - ./concerto/config/routing_custom.yml:/usr/src/concerto/app/config/routing_custom.yml
        - ./concerto/config/test_base.html.twig:/usr/src/concerto/app/Resources/views/test_base.html.twig
        - ./concerto/R:/usr/src/concerto/R
        - ./concerto/rcache:/usr/src/concerto/src/Concerto/PanelBundle/Resources/public/rcache
        - ./concerto/files:/usr/src/concerto/src/Concerto/PanelBundle/Resources/public/files
        - ./concerto/favicon.ico:/usr/src/concerto/web/favicon.ico
        - ./concerto/php-fpm/php-fpm.conf:/usr/local/etc/php-fpm.conf
        - ./concerto/php-fpm/www.conf:/usr/local/etc/php-fpm.d/www.conf
        - ./concerto/php/php.ini:/usr/local/etc/php/php.ini
        - ./concerto/nginx/nginx.conf:/etc/nginx/nginx.conf
        - ./concerto/nginx/sites/concerto.conf:/etc/nginx/sites-available/concerto.conf
        - ./logs/concerto/nginx:/var/log/nginx
        - ./logs/concerto/php-fpm:/var/log/php-fpm
        - ./logs/concerto/app:/usr/src/concerto/var/logs
        - ./logs/concerto/sessions:/usr/src/concerto/src/Concerto/TestBundle/Resources/sessions
      ports:
        - "80:80"
```

5. 进入文件夹：`usr/concerto/doker-compose`

6. 最后使用使用`docker-compose`命令启动容器
```
sudo docker-compose up -d
```

此外，除了使用代码安装，还可以首先安装portainer操作界面，进行可视化安装。
```
$ docker volume create portainer_data
$ docker run -d -p 8000:8000 -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```
