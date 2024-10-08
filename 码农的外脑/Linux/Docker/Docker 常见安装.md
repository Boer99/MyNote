
## MySQL

```bash
# 简洁版
docker run -d -p 3307:3306 --privileged=true \
-e MYSQL_ROOT_PASSWORD=123456 \
--name mysql5_7 \
mysql:5.7

docker run -d -p 3306:3306 --privileged=true \
-v /boerdata/mysql8_0_26/log:/var/log/mysql \
-v /boerdata/mysql8_0_26/data:/var/lib/mysql \
-v /boerdata/mysql8_0_26/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
--name mysql8_0_26 \
mysql:8.0.26

docker run -d -p 3306:3306 --privileged=true -v /zzyyuse/mysql/log:/var/log/mysql -v /zzyyuse/mysql/data:/var/lib/mysql -v /zzyyuse/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456  --name mysql mysql:8.0.26
```

# Nacos

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