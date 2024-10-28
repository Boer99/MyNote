# Docker镜像配置

```
{
  "registry-mirrors": [
        "https://h3o3bwt2.mirror.aliyuncs.com",
        "https://docker.registry.cyou",
        "https://docker-cf.registry.cyou",
        "https://dockercf.jsdelivr.fyi",
        "https://docker.jsdelivr.fyi",
        "https://dockertest.jsdelivr.fyi",
        "https://mirror.aliyuncs.com",
        "https://dockerproxy.com",
        "https://mirror.baidubce.com",
        "https://docker.m.daocloud.io",
        "https://docker.nju.edu.cn",
        "https://docker.mirrors.sjtug.sjtu.edu.cn",
        "https://docker.mirrors.ustc.edu.cn",
        "https://mirror.iscas.ac.cn",
        "https://docker.rainbond.cc"
        ]
}
```


## MySQL

[Docker实操：安装MySQL5.7详解（保姆级教程）_docker 安装mysql5.7-CSDN博客](https://blog.csdn.net/u012263509/article/details/142323103)

拉取镜像
```
docker pull mysql:5.7
```

创建目录
```bash
mkdir -p /home/service/mysql/logs \
mkdir -p /home/service/mysql/data \
mkdir -p /home/service/mysql/conf
```

新建配置文件（）
```bash
cd /home/service/mysql/conf
touch my.cnf
vim my.cnf

[mysqld]
user=mysql
character-set-server=utf8
default_authentication_plugin=mysql_native_password
default-time_zone='+8:00'
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
```

创建容器并启动
- 默认密码
```bash
# 简洁版
docker run -d -p 3307:3306 --privileged=true \
-e MYSQL_ROOT_PASSWORD=gong990117 \
--name mysql5_7 \
mysql:5.7

# 完整版
docker run -d -p 3306:3306 \
--name mysql57 \
-v /home/service/mysql/logs:/var/log/mysql \
-v /home/service/mysql/data:/var/lib/mysql \
-v /home/service/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=gong990117 \
--restart=always \
mysql:5.7
```

如果容器启动就退出了，查看日志，出现报错：`mysqld: Can't read dir of '/etc/mysql/conf.d/' (Errcode: 2 - No such file or directory)`，可以增加如下文件：

```
mkdir -p /home/service/mysql/conf/conf.d \
mkdir -p /home/service/mysql/conf/mysql.conf.d
```

云服务器开放服务器安全组中 3306 端口

![](../../assets/Pasted%20image%2020241008215236.png)

# Nacos

拉取镜像
```
docker pull nacos/nacos-server
```

创建并运行容器
```bash
docker run -d \
--name nacos \
--privileged \
--cgroupns host \
--env JVM_XMX=256m \
--env JVM_XMS=256m \
--env MODE=standalone \
-p 8848:8848/tcp \
-p 9848:9848/tcp \
--restart=always \
-w /home/nacos \
nacos/nacos-server
```

# Jenkins

创建目录
```bash
mkdir -p /home/jenkins/jenkins_home
chmod 777 /home/jenkins/jenkins_home
```

创建并运行容器
```bash
docker run -d \
--privileged \
--name jenkins \
-p 8080:8080 \
-p 50000:50000 \
-v /home/jenkins/jenkins_home:/var/jenkins_home \
jenkins/jenkins:lts
```

启动端口 8080，通过 `docker logs xxx` 查看密码


# Redis

官方的配置文件地址：[Redis configuration | Docs](https://redis.io/docs/latest/operate/oss_and_stack/management/config/)

创建挂载目录
```
mkdir -p /home/redis/data
```

在 `/home/redis` 下创建配置文件 redis.conf `touch redis.conf`

官方的配置文件地址：[Redis configuration | Docs](https://redis.io/docs/latest/operate/oss_and_stack/management/config/)，按照对应版本复制到 redis.conf 

需要修改的配置
```conf
# bind 127.0.0.1 -::1
# 直接注释掉或者改成下面的都可以，上面的是只允许本机访问
bind 0.0.0.0

# 设置密码
requirepass gong990117
```

创建并运行容器
```bash
docker run \
--restart=always \
-p 6379:6379 \
--name redis74 \
-v /home/redis/redis.conf:/etc/redis/redis.conf \
-v /home/redis/data:/data \
-d redis redis-server /etc/redis/redis.conf \
-- appendonly yes
```


# xxl-job

数据库准备 [doc/db/tables_xxl_job.sql · 许雪里/xxl-job - Gitee.com](https://gitee.com/xuxueli0323/xxl-job/blob/master/doc/db/tables_xxl_job.sql)

拉取镜像
```bash
docker pull xuxueli/xxl-job-admin:2.4.0
```

创建挂载目录
```
mkdir -p /home/xxl-job/logs
```

创建并启动镜像
- 数据库连接的用户名和密码
```bash
docker run -d \
-p 8088:8088 \
-v /home/xxl-job/logs:/data/applogs \
-v /home/xxl-job/application.properties:/xxl-job/xxl-job-admin/src/main/resources/application.properties \
-e PARAMS="--server.port=8088 \
--spring.datasource.url=jdbc:mysql://116.198.253.55:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai \
--spring.datasource.username=root \
--spring.datasource.password=gong990117" \
--name xxl-job-admin \
xuxueli/xxl-job-admin:2.4.0
```

访问 http://116.198.253.55:8088/xxl-job-admin

默认用户名密码：root，123455

# Nginx

https://blog.csdn.net/BThinker/article/details/123507820

创建挂载目录
```
mkdir -p /home/nginx/conf \
mkdir -p /home/nginx/log \
mkdir -p /home/nginx/html
```

==容器中的 nginx.conf 文件和 conf.d 文件夹复制到宿主机！==
```bash
# 生成容器
docker run --name nginx -p 80:80 -d nginx
# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /home/nginx/conf/nginx.conf
# 将容器conf.d文件夹下内容复制到宿主机
docker cp nginx:/etc/nginx/conf.d /home/nginx/conf/conf.d
# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /home/nginx/
```

创建并运行容器。==这里要注意：反向代理的端口号定要端口映射，例如你要监听宿主机的8888端口，就要 `-p 8888:8888`，不然 nginx 容器根本不知道外面访问了 8888 端口==
```bash
# 一般前端项目都会用80端口，默认端口是不需要显式地指定的
docker run \
-p 80:80 \
--name nginx \
-v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/nginx/log:/var/log/nginx \
-v /home/nginx/html:/usr/share/nginx/html \
-d nginx:latest

# 8888示例
docker run \
-p 80:80 \
-p 8888:8888 \
--name nginx \
-v /home/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /home/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /home/nginx/log:/var/log/nginx \
-v /home/nginx/html:/usr/share/nginx/html \
-d nginx:latest
```



