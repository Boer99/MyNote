# Docker介绍
## Docker的基本组成
# Docker安装
## Centos7安装与卸载Docker
[https://docs.docker.com/engine/install/centos](https://docs.docker.com/engine/install/centos)<br />确定是Centos7及以上版本
```
cat etc/redhat-release
```
卸载旧版本
```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
3、yum安装GCC相关
```
yum -y install gcc
yum -y install gcc-c++
```
4、安装需要的软件包
```
yum install -y yum-utils
```
**5、设置stable镜像仓库**<br />**官网给的仓库是国外，我们要设置国内的**
```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
6、更新yum软件包索引
```
yum makecache fast
```
7、安装Docker CE
```
yum -y install docker-ce docker-ce-cli containerd.io
```
8、启动Docker
```
systemctl start docker
```
9、测试
```
docker version
docker run hello-world
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693297201445-cb695cdc-4f2f-462e-b211-19e93fb749a5.png#averageHue=%23d4cbb8&clientId=u17106892-b3d9-4&from=paste&height=709&id=udfae243d&originHeight=1116&originWidth=2233&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=109749&status=done&style=none&taskId=ufc170474-3f36-43fe-af9a-70488875692&title=&width=1417.7778421634118)<br />10、卸载<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693297243740-f2931f53-4c97-45c4-b315-5c1120b1aa13.png#averageHue=%23cbcd9b&clientId=u17106892-b3d9-4&from=paste&height=397&id=u87866bc0&originHeight=625&originWidth=1438&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=70838&status=done&style=none&taskId=u810fdf6c-ec11-472f-95c2-fc591d99f5b&title=&width=913.0159144787219)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693297559992-756f0add-c523-45e4-b98f-558581dac912.png#averageHue=%23f9f8f7&clientId=u17106892-b3d9-4&from=paste&height=164&id=ua16983b2&originHeight=258&originWidth=1070&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=31174&status=done&style=none&taskId=u959d02b6-c87d-4db8-b628-7cd7983eecf&title=&width=679.3651102171297)
## 阿里云镜像加速
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693297826701-1b76e558-bc4c-44a4-9093-5a1a3c087694.png#averageHue=%23f8f7f6&clientId=u17106892-b3d9-4&from=paste&height=486&id=ub9c524b1&originHeight=765&originWidth=1545&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=133734&status=done&style=none&taskId=u033aa1ba-ebd8-4740-ad65-808b8712c40&title=&width=980.952425500435)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693297831567-6a829ff6-27b7-42b0-9260-66fb61f02130.png#averageHue=%23fba38c&clientId=u17106892-b3d9-4&from=paste&height=683&id=u170003d0&originHeight=1075&originWidth=1561&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=126710&status=done&style=none&taskId=u02042d83-da79-4f7b-a68f-247f9d6914a&title=&width=991.1111561205041)
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://h3o3bwt2.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
## 永远的helloworld
`docker run hello-world`都做了什么<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693297875905-041bc395-238e-474c-a94d-f1ce74b9d9ce.png#averageHue=%23f9f9f9&clientId=u17106892-b3d9-4&from=paste&height=460&id=u824ff8b8&originHeight=724&originWidth=1480&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=113742&status=done&style=none&taskId=u7095414a-a665-4f7a-b6b3-4f568273ada&title=&width=939.6825823564037)
## Docker为什么比VM虚拟机快
(1) docker有着比虚拟机更少的抽象层<br />由于docker不需要Hypervisor(虚拟机)实现硬件资源虚拟化,运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有明显优势。<br />(2) docker利用的是宿主机的内核,而不需要加载操作系统OS内核<br />当新建一个容器时,docker不需要和虚拟机一样重新加载一个操作系统内核。进而避免引寻、加载操作系统内核返回等比较费时费资源的过程,当新建一个虚拟机时,虚拟机软件需要加载OS,返回新建过程是分钟级别的。而docker由于直接利用宿主机的操作系统,则省略了返回过程,因此新建一个docker容器只需要几秒钟。<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693297980064-7c7a6b6e-37e1-4e9c-838c-04c98f621ed0.png#averageHue=%235c96b2&clientId=u17106892-b3d9-4&from=paste&height=569&id=ub03d938c&originHeight=986&originWidth=1040&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=471218&status=done&style=none&taskId=uc3fbeb2f-8e1c-4c47-a6af-3feec3d5d1a&title=&width=600.2886151582064)
# Docker常用命令
## 帮助启动类命令
```
启动docker: systemctl start docker

停止docker:systemctl stop docker

重启docker: systemctl restart docker

查看docker状态: systemctl status docker

开机启动: systemctl enable docker

查看docker概要信息: docker info

查看docker总体帮助文档: docker --help

查看docker命令帮助文档: docker 具体命令 --help
```
查看docker总体帮助文档，可以看到所有的docker命令，然后再通过 `docker 具体命令 --help` 查看某个具体命令的帮助文档<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298099314-058179db-cc41-48b1-9e7e-93342b5c6540.png#averageHue=%23d4cbb8&clientId=u17106892-b3d9-4&from=paste&height=644&id=u86c5245a&originHeight=1116&originWidth=1331&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=87250&status=done&style=none&taskId=ucb28a79a-b290-4768-8fd8-aa0940adf16&title=&width=768.2539872842045)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298110129-98a6deba-9ef8-4a14-9c00-b9043c3ef255.png#averageHue=%23d4cbb9&clientId=u17106892-b3d9-4&from=paste&height=474&id=u9f556703&originHeight=822&originWidth=1969&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=72584&status=done&style=none&taskId=u77aa6992-1108-4df7-a234-09bfb6147b7&title=&width=1136.507964660104)
## 镜像命令
### `docker images [OPTIONS]`
列出本地主机上的镜像<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298166599-0dcc8101-8082-45ac-936f-8bf27938f5ad.png#averageHue=%23d4cab8&clientId=u17106892-b3d9-4&from=paste&height=74&id=u832be026&originHeight=129&originWidth=1120&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=14836&status=done&style=none&taskId=u6de79856-440f-4a57-84f8-2ef512e419b&title=&width=646.4646624780684)<br />各个选项说明：

- REPOSITORY：表示镜像的仓库源
- TAG：镜像的标签版本号
   - 同一仓库源可以有多个 TAG版本，代表这个仓库源的不同个版本，我们使用 `REPOSITORY:TAG` 来定义不同的镜像。
   - 如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像
- IMAGE ID：镜像ID
- CREATED：镜像创建时间
- SIZE：镜像大小

options说明：

- -a：列出本地所有的镜像（含历史镜像层）
- -q：只显示历史镜像
### `docker search [OPTIONS]` 镜像名字
查找某个镜像<br />[https://hub.docker.com](https://hub.docker.com)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298321919-9c7ec11e-98d1-4fb3-8bcb-3fcc9f793be1.png#averageHue=%23d4cbb8&clientId=u17106892-b3d9-4&from=paste&height=692&id=u680087b6&originHeight=1199&originWidth=2233&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=158676&status=done&style=none&taskId=ue03f690a-9ce6-499c-b59f-343e83e1f36&title=&width=1288.8889208156488)<br />OPTIONS说明：

- `--limit N`：只列出前N个

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298347681-e8a2929c-41a8-4957-8d11-fc836f579961.png#averageHue=%23d4cab8&clientId=u17106892-b3d9-4&from=paste&height=173&id=u4cdc8272&originHeight=300&originWidth=1890&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=42014&status=done&style=none&taskId=u0c0d2a85-5fcc-45e2-9fe1-082352dc44f&title=&width=1090.9091179317404)
### `docker pull 镜像名字 [:TAG]`
下载镜像<br />TAG表示版本号，例如 `docker pull mysql:5.7`<br />没有TAG就等价于  `docker pull 镜像名字:latest`<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298383718-1e8129e9-8347-49bc-9db5-590b84447281.png#averageHue=%23d4cab8&clientId=u17106892-b3d9-4&from=paste&height=383&id=u7556c334&originHeight=664&originWidth=1436&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=83010&status=done&style=none&taskId=ud711c4b7-00f7-4a6a-be47-03e598731a9&title=&width=828.8600493915234)
### `docker system df`
查看镜像/容器/数据卷所占用空间<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298404176-a6de6946-61a3-4331-803a-9f178023fc33.png#averageHue=%23d4cab8&clientId=u17106892-b3d9-4&from=paste&height=148&id=uf8272550&originHeight=257&originWidth=1057&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=23250&status=done&style=none&taskId=u276f19e1-3517-4031-ba0d-d3d9664f2f7&title=&width=610.1010252136771)
### 删除镜像
```
docker rmi 镜像名/ID
```
还存在容器，不能直接删镜像，强制删除
```
docker rmi -f 镜像名/ID
```
删除多个
```
docker rmi -f 镜像1:TAG 镜像2:TAG
```
删除全部
```
docker rmi -f $(docker images -qa)
```
### 虚悬镜像
仓库名、标签名都是 `<none>` 的镜像，俗称虚悬镜像`dangling image`<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298707025-8755a0c3-f328-43f9-b792-a7e9087c6bd2.png#averageHue=%23efe8e9&clientId=u17106892-b3d9-4&from=paste&height=92&id=u98e4c429&originHeight=160&originWidth=1471&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=83395&status=done&style=none&taskId=u23ab81ce-7d81-41fd-be8f-a8a62964f2d&title=&width=849.062070093963)
## 容器命令
> 通过ubuntu演示，因为体积小，docker pull ubuntu

### 新建+启动命令
```
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
OPTIONS说明：

- --name="容器新名字为容器指定一个名称
- -d: 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)
- -i: 以交互模式运行容器，通常与 t 同时使用
- -t: 为容器重新分配一个伪输入终端，通常与-i 同时使用等待交互)也即启动交互式容器(前台有伪终端，
- -P: 随机端口映射，大写P
- -p: 指定端口映射，小写p

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298848087-df1719a7-660f-4069-ba4a-0e1c44b4123b.png#averageHue=%23d0dcb8&clientId=u17106892-b3d9-4&from=paste&height=242&id=uea142c98&originHeight=419&originWidth=1307&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=200644&status=done&style=none&taskId=u760b1d30-a809-4718-b051-885e25decbf&title=&width=754.4011730882459)<br />使用镜像ubuntu:latest以交互模式启动一个容器,在容器内执行/bin/bash命令
```
docker run -it ubuntu /bin/bash
```
要退出终端，直接输入 exit
### 列出当前所有正在运行的容器
```
docker ps [OPTIONS]
```
[OPTIONS]说明

- -a：列出当前所有正在运行的容器+历史上运行过的
- -l：显示最近创建的容器
- -n：显示最近n个创建的容器
- -q：静默模式，只显示容器编号.

新建并运行一个容器<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298890209-d9ffe699-7fa3-41f9-bc0e-63c416d5ac6f.png#averageHue=%23d4cab7&clientId=u17106892-b3d9-4&from=paste&height=53&id=uec4c9a98&originHeight=91&originWidth=931&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=12020&status=done&style=none&taskId=u3a34807f-8357-4a25-b6de-785e8ba30df&title=&width=537.3737506848944)<br />打开一个新的终端窗口查看正在运行的容器，名字是随机的<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693298895718-65969743-a17a-4fb4-a583-f179fae584ff.png#averageHue=%23d4cab8&clientId=u17106892-b3d9-4&from=paste&height=77&id=u5a2a77be&originHeight=133&originWidth=1690&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=17279&status=done&style=none&taskId=u7b40d99d-ef69-4c30-9dce-4d778c83409&title=&width=975.4689996320853)
### 退出容器
第一种：run进去，exit退出，容器停止运行<br />第二种：run进去，ctrl+p+q退出，容器不停止
### 启动已停止运行的容器
```
docker start 容器id或容器名
```
先docker ps -a查看容器的状态
### 重启容器
```
docker restart 容器id或容器名
```
### 停止容器
```
docker stop 容器id或容器名
```
### 强制停止容器
```
docker kill 容器id或容器名
```
### 删除已停止容器
删除单个
```
docker rm 容器ID
docker rm -f 容器ID  # 强删，没停止的也删
```
全删
```
docker rm -f $(docker ps -a -q)
docker ps -a -q | xargs docker rm
```

### 启动守护式容器（docker的后台服务器）
在大部分场景下我们希望docker的服务是在后台运行的，可以通过-d指定容器的后台运行模式
```
docker run -d 容器名
```

使用镜像centos:latest以后台模式启动一个容器：docker run -d centos

问题：然后docker ps -a 进行查看, 会发现容器已经退出<br />很重要的要说明的一点：**Docker容器后台运行，就必须有一个前台进程**<br />容器运行的命令如果不是那些一直挂起的命令（比如运行top，tail），就是会自动退出的。<br /> <br />这个是docker的机制问题,比如你的web容器,我们以nginx为例，正常情况下,<br />我们配置启动服务只需要启动响应的service即可。例如service nginx start<br />但是,这样做,nginx为后台进程模式运行,就导致docker前台没有运行的应用,<br />这样的容器后台启动后,会立即自杀因为他觉得他没事可做了.<br />所以，**最佳的解决方案是，将你要运行的程序以前台进程的形式运行**<br />常见就是命令行模式，表示我还有交互操作，别中断，O(∩_∩)O哈哈~

redis前后台演示
```
docker run -it redis:6.0.8
docker run -d redis:6.0.8
```

### 查看容器日志
```
docker logs 容器ID
```
### 查看容器内运行的进程
```
docker top 容器ID
```
### 查看容器内部细节
```
docker inspect 容器ID
```
### 进入正在运行的容器并以命令行交互
```
docker exec -it 容器ID bashShell
docker attach 容器ID
```
两种方式的区别：

- **exec（推荐）**是在容器中打开新的终端，并且可以启动新的进程。用exit退出不会导致容器的停止
- attach直接进入容器启动命令的终端，不会启动新的进程。用exit退出会导致容器的停止
### 从容器内拷贝文件到主机上
```
docker cp 容器ID:容器内路径 目的主机路径
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693299319193-6240a403-c25c-45c2-888b-d08991781ae7.png#averageHue=%23f3eded&clientId=u17106892-b3d9-4&from=paste&height=201&id=u61a7dc70&originHeight=348&originWidth=1896&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=155222&status=done&style=none&taskId=ud438757f-de22-474d-a2d3-362d12a0cf6&title=&width=1094.37232148073)
### 导出容器为tar和导入tar为镜像
export 导出容器的内容留作为一个tar归档文件[对应import命令]
```
docker export 容器ID > 文件名.tar.gz
```
import 从tar包中的内容创建一个新的文件系统再导入为镜像[对应export]
```
cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号
```
导出案例<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693299467576-0d0264ab-71b9-4c3a-b752-412dd63498f6.png#averageHue=%23fdfafa&clientId=u17106892-b3d9-4&from=paste&height=232&id=u840026a5&originHeight=402&originWidth=1654&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=246219&status=done&style=none&taskId=u161dc381-af7c-4b91-91a7-105cfdbe455&title=&width=954.6897783381474)<br />导入案例<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693299472545-7c34ac1d-9575-49c3-bd18-38e7497e9b75.png#averageHue=%23d4cab7&clientId=u17106892-b3d9-4&from=paste&height=379&id=ud9be3c2a&originHeight=657&originWidth=1313&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=89765&status=done&style=none&taskId=u74efac07-eb49-45a7-9418-1be3c5b1e88&title=&width=757.8643766372355)
# Docker镜像
## 是什么？
**镜像**是一种轻量级、可执行的独立软件包，它包含运行某个软件所需的所有内容，我们把应用程序和配置依赖打包好形成一个可交付的运行环境(包括代码、运行时需要的库、环境变量和配置文件等)，这个打包好的运行环境就是image镜像文件。<br />只有通过这个镜像文件才能生成Docker容器实例(类似Java中new出来一个对象)。
## 分层的镜像
以我们的pull为例，在下载的过程中我们可以看到docker的镜像好像是在一层一层的在下载<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693299980108-f37a9ad8-4a0e-477a-9879-d58c65d2ab09.png#averageHue=%23f4f3f3&clientId=u17106892-b3d9-4&from=paste&height=446&id=fwBRJ&originHeight=772&originWidth=1243&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=254587&status=done&style=none&taskId=u54140021-e932-431c-b60c-49f299b279e&title=&width=717.4603352323563)
## UnionFS（联合文件系统）
Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。<br />**特性：**一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录
## Docker镜像加载原理
docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。<br />`bootfs(boot file system)`主要包含bootloader和kernel，bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是引导文件系统bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

`rootfs (root file system)`，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。

平时我们安装进虚拟机的CentOS都是好几个G，为什么docker这里才200M？？<br />对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了，因为底层直接用Host的kernel，自己只需要提供 rootfs 就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别, 因此不同的发行版可以公用bootfs。
## 为什么Docker镜像要采用分层结构呢？
镜像分层最大的一个好处就是共享资源，方便复制迁移，就是为了复用。<br />比如说有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；<br />同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。
## 镜像层、容器层
Docker镜像层都是只读的，容器层是可写的<br />当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。<br />所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中。只有容器层是可写的，容器层下面的所有镜像层都是只读的。
## Docker镜像commit操作
docker commit提交容器副本使之成为一个新的镜像
```
docker commit -m="提交的描述信息” -a="作者” 容器ID 要创建的目标镜像名:[标签名]
```

---

**案例演示ubuntu安装vim**<br />原始的ubuntu是默认不带vim的<br />在ubuntu容器内安装vim，容器内执行下述两条命令：
```
apt-get update
apt-get -y install vim
```
安装成功后commit新镜像<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693300227619-f122f239-c766-43c9-bf9d-b38ef92ab88b.png#averageHue=%23fef7f7&clientId=u17106892-b3d9-4&from=paste&height=187&id=u7a7ccc01&originHeight=324&originWidth=1792&originalType=binary&ratio=1.5749999284744263&rotation=0&showTitle=false&size=263134&status=done&style=none&taskId=u8a129018-1cb7-41aa-bb66-4ead044af13&title=&width=1034.3434599649095)
## 小总结
Docker中的镜像分层，支持通过扩展现有镜像，创建新的镜像。类似Java继承于一个Base基础类，自己再按需扩展。<br />新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层
# 本地镜像发布到阿里云
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693300716050-ea73c1ea-2523-4814-832a-1cfd0b9cd33c.png#averageHue=%23f2f2f2&clientId=u7cbccd17-ee37-4&from=paste&height=518&id=qnIuL&originHeight=897&originWidth=840&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=194642&status=done&style=none&taskId=uae7512eb-61c2-4ca3-b846-ddce46116b8&title=&width=484.8484968585513)
## 创建镜像仓库
进入容器镜像服务，选择个人实例<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693300830333-e09c3d70-9acd-4a9a-b218-d2f5e14c7336.png#averageHue=%23fcfaf9&clientId=u7cbccd17-ee37-4&from=paste&height=726&id=ucd875358&originHeight=1258&originWidth=2560&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=128191&status=done&style=none&taskId=uf91eeb9c-6b70-47f1-9dbb-3ee59bc8d62&title=&width=1477.633514235585)<br />创建命名空间<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693300835983-752ca58f-f3c3-475f-83d8-86546fa53ccc.png#averageHue=%23fdfdfc&clientId=u7cbccd17-ee37-4&from=paste&height=726&id=ubde197f8&originHeight=1258&originWidth=2560&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=91877&status=done&style=none&taskId=ufa7fd96c-1f14-4946-af18-a4d670053e6&title=&width=1477.633514235585)<br />创建镜像仓库<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693300850709-8acb999d-51d6-47ab-8b6a-196b6107d33c.png#averageHue=%23fcfcfc&clientId=u7cbccd17-ee37-4&from=paste&height=629&id=u9a5f6662&originHeight=1089&originWidth=1349&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=59591&status=done&style=none&taskId=u6caaec91-6315-45ec-b8f2-ab894f37017&title=&width=778.6435979311735)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693300880056-8748f621-8aca-421d-949f-d32e95f2026d.png#averageHue=%23fcfbfb&clientId=u7cbccd17-ee37-4&from=paste&height=347&id=u94eac9a1&originHeight=601&originWidth=1384&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=41828&status=done&style=none&taskId=u1c8ae18e-9f19-4125-967a-b94f64bb827&title=&width=798.8456186336131)<br />进入管理界面获得脚本<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693300887833-75d2fc00-1bb8-47e2-83e5-1e2bbfb9f04b.png#averageHue=%23fdfcfc&clientId=u7cbccd17-ee37-4&from=paste&height=726&id=ua4d393e8&originHeight=1258&originWidth=2560&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=105559&status=done&style=none&taskId=uc1075439-e273-49dc-94d2-5f74a7233e5&title=&width=1477.633514235585)
## 推送和下载镜像
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693300964460-0e8ed882-24be-4400-bfe3-45bf806a5efb.png#averageHue=%23f9f5f4&clientId=u7cbccd17-ee37-4&from=paste&height=726&id=u74c9da6a&originHeight=1258&originWidth=2560&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=309070&status=done&style=none&taskId=u19428399-4f15-4aea-8076-ed2b00533d3&title=&width=1477.633514235585)<br />登录阿里云Docker Registry
```
docker pull registry.cn-hangzhou.aliyuncs.com/boerspace/myubuntu1.3:[镜像版本号]
```
推送
```
docker login --username=gboer registry.cn-hangzhou.aliyuncs.com
docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/boerspace/myubuntu1.3:[镜像版本号]
docker push registry.cn-hangzhou.aliyuncs.com/boerspace/myubuntu1.3:[镜像版本号]
```
下载
> 下载不用登录阿里云Docker Registry

```
docker pull registry.cn-hangzhou.aliyuncs.com/boerspace/myubuntu1.3:[镜像版本号]
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693301029693-48aaca56-bf1a-4268-8ea2-64ce34bb9b34.png#averageHue=%23fefbfb&clientId=u7cbccd17-ee37-4&from=paste&height=346&id=u576136f2&originHeight=600&originWidth=1800&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=434256&status=done&style=none&taskId=ub86e43c7-a29b-4df7-8294-a40b9023e77&title=&width=1038.9610646968956)
# 本地镜像发布到私有库
暂时跳过
# Docker容器数据卷
## 坑，容器卷记得加入！
Docker挂载主机目录访问如果出现cannot open directory .: Permission denied<br />解决办法：在挂载目录后多加一个 `**--privileged=true**` 参数即可<br />如果是CentOS7安全模块会比之前系统版本加强，不安全的会先禁止，所以目录挂载的情况被默认为不安全的行为，<br />在SELinux里面挂载目录被禁止掉了额，如果要开启，我们一般使用`--privileged=true`命令，扩大容器的权限解决挂载目录没有权限的问题，也即<br />使用该参数，**container内的root拥有真正的root权限，否则，container内的root只是外部的一个普通用户权限**
## 介绍
**1、是什么？**<br />卷就是目录或文件，存在于一个或多个容器中，由docker挂载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性：<br />卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷<br />类似于Redis里的rdb和aof文件<br />作用：**将docker容器内的数据保存进宿主机的磁盘**

---

**2、能干嘛？**<br />将运用与运行的环境打包镜像，run后形成容器实例运行 ，但是我们对数据的要求希望是持久化的<br />Docker容器产生的数据，如果不备份，那么当容器实例删除后，容器内的数据自然也就没有了。<br />为了能保存数据在docker中我们使用卷。<br />特点：<br />1：**数据卷可在容器之间共享或重用数据**<br />2：卷中的更改可以直接实时生效，爽<br />3：数据卷中的更改不会包含在镜像的更新中<br />4：数据卷的生命周期一直持续到没有容器使用它为止
## 数据卷案例
### 宿主和容器之间印射添加容器卷
#### 命令
```
docker run -it -v /宿主机目录:/容器内目录 镜像名 /bin/bash
例：
docker run -it --name myu3 --privileged=true -v /tmp/myHostData:/tmp/myDockerData ubuntu /bin/bash
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693301372623-87a5e778-e6c9-4756-8666-262bfab41f90.png#averageHue=%23f9f2f2&clientId=u7cbccd17-ee37-4&from=paste&height=449&id=u84141473&originHeight=778&originWidth=1313&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=208991&status=done&style=none&taskId=u71298bc9-42c6-46b3-a74b-0b768be81f3&title=&width=757.8643766372355)
#### 查看数据卷是否挂载成功
通过查看容器内部细节命令
```
docker inspect 容器ID
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693301449460-c40b0f80-48cc-459f-a3f0-062296f631f0.png#averageHue=%23f9f2f2&clientId=u7cbccd17-ee37-4&from=paste&height=449&id=u7c00b30c&originHeight=778&originWidth=1313&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=208991&status=done&style=none&taskId=ue634a53a-990a-447a-a1b7-1c62fa9cca8&title=&width=757.8643766372355)
#### 容器和宿主机之间数据共享

1. docker修改，主机同步获得
2. 主机修改，docker同步获得
3. docker容器stop，主机修改，docker容器重启看数据是否同步。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693301517656-36d2a12c-2218-476e-add9-10f0d0cb5dd5.png#averageHue=%23fbf4f3&clientId=u7cbccd17-ee37-4&from=paste&height=390&id=u07d16e65&originHeight=675&originWidth=2086&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=433894&status=done&style=none&taskId=u9f06657b-aafa-4a60-bb25-5363adab144&title=&width=1204.0404338654025)
### 读写规则映射添加说明
> 通俗的说，就是设置容器的读写权限，例如ro就是容器只能对数据卷进行读操作，不能进行写操作

默认规则：rw，即可读可写
```
docker run -it -v /宿主机目录:/容器内目录:rw 镜像名 /bin/bash
```
只读规则：ro，容器实例内部被限制，只能读不能写。宿主机写入内容可以同步给容器内
```
docker run -it -v /宿主机目录:/容器内目录:ro 镜像名 /bin/bash
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693301574924-2e7eafcd-0e23-42fa-abc2-953a123a4e4a.png#averageHue=%23fef8f8&clientId=u7cbccd17-ee37-4&from=paste&height=241&id=ua44653a3&originHeight=417&originWidth=1478&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=189517&status=done&style=none&taskId=uecb0aef0-1e01-4815-8a7a-41976185bc1&title=&width=853.102474234451)
### 卷的继承和共享
```
docker run -it --privileged=true --volumes-from 父类 --name 容器名 镜像
```
案例：容器u2继承容器u1的卷规则
```
docker run -it --privileged=true --volumes-from u1 --name u2 ubuntu
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693301615492-264318cf-d024-446d-91f6-872d7fa22688.png#averageHue=%23fef9f9&clientId=u7cbccd17-ee37-4&from=paste&height=235&id=u506524cf&originHeight=407&originWidth=1412&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=182441&status=done&style=none&taskId=ua7fd9921-402c-4dcc-bac4-d40346d7341&title=&width=815.0072351955648)
# Docker常规安装
## 总体步骤

1. 搜索镜像
2. 拉取镜像
3. 查看镜像
4. 启动镜像（服务端口映射）
5. 停止容器
6. 删除容器
## 安装tomcat
> 有些问题

## 安装mysql
搜索镜像
```
docker search mysql
```
拉取镜像
```
docker pull mysql:5.7
```
新建运行容器（端口印射、容器卷） 密码：
```
docker run -d -p 3306:3306 --privileged=true -v /zzyyuse/mysql/log:/var/log/mysql -v /zzyyuse/mysql/data:/var/lib/mysql -v /zzyyuse/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456  --name mysql mysql:5.7
```
宿主机新建my.conf，通过容器卷同步给容器<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/12496339/1693301835133-c43061e2-957e-460e-ae16-898a356bd16d.png#averageHue=%23fefbfb&clientId=u7cbccd17-ee37-4&from=paste&height=250&id=ud6311c9d&originHeight=433&originWidth=1183&originalType=binary&ratio=1.7324999570846558&rotation=0&showTitle=false&size=111862&status=done&style=none&taskId=udfac2487-2416-417a-9d4d-361ecc5bc35&title=&width=682.8282997424598)
```
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8
```
重启容器进入，查看mysql的字符编码
