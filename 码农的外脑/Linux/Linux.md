# Linux目录结构
## 概述
![](https://cdn.nlark.com/yuque/0/2023/jpeg/12496339/1679640311579-0538a6dd-9ec8-482f-85d8-1031f0c40647.jpeg)

<font style="color:rgb(0,0,0);">linux 的文件系统是采用级层式的树状目录结构，在此结构中的最上层是根目录“/”，然后在此目录下再创建其他的目录。</font>

<font style="color:rgb(0,0,0);background-color:#FBDE28;">在 Linux 世界里，一切皆文件(!!)</font>

## <font style="color:rgb(0,0,0);">具体目录结构解析</font>
+ **<font style="color:rgb(51, 51, 51);">/bin</font>**<font style="color:rgb(51, 51, 51);">：bin 是 Binaries (二进制文件) 的缩写, 这个目录存放着最经常使用的命令。</font>
+ **<font style="color:rgb(51, 51, 51);">/sbin</font>**<font style="color:rgb(51, 51, 51);">：s 就是 Super User 的意思，是 Superuser Binaries (超级用户的二进制文件) 的缩写，这里存放的是系统管理员使用的系统管理程序。</font>
+ **<font style="color:rgb(51, 51, 51);">/</font>****<font style="color:rgb(51, 51, 51);background-color:#FBDE28;">home</font>**<font style="color:rgb(51, 51, 51);">：用户的主目录，在 Linux 中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的，如上图中的 alice、bob 和 eve。</font>
+ **<font style="color:rgb(51, 51, 51);">/root</font>**<font style="color:rgb(51, 51, 51);">：该目录为系统管理员，也称作超级权限者的用户主目录。</font>
+ **<font style="color:rgb(51, 51, 51);">/lib</font>**<font style="color:rgb(51, 51, 51);">：lib 是 Library(库) 的缩写这个目录里存放着系统最基本的动态连接共享库，其作用类似于 Windows 里的 DLL 文件。几乎所有的应用程序都需要用到这些共享库。</font>
+ **<font style="color:rgb(51, 51, 51);">/lost+found</font>**<font style="color:rgb(51, 51, 51);">：这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。</font>
+ **<font style="color:rgb(51, 51, 51);">/</font>****<font style="color:rgb(51, 51, 51);background-color:#FBDE28;">etc</font>****<font style="color:rgb(51, 51, 51);">：</font>**<font style="color:rgb(51, 51, 51);">etc 是 Etcetera(等等) 的缩写,这个目录用来存放所有的系统管理所需要的配置文件和子目录。</font>
+ **<font style="color:rgb(51, 51, 51);">/</font>****<font style="color:rgb(51, 51, 51);background-color:#FBDE28;">usr</font>**<font style="color:rgb(51, 51, 51);">：usr 是 unix shared resources(共享资源) 的缩写，这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于 windows 下的 program files 目录。</font>
+ **<font style="color:rgb(51, 51, 51);">/proc【不能动】</font>**<font style="color:rgb(51, 51, 51);">：proc 是 Processes(进程) 的缩写，/proc 是一种伪文件系统（也即虚拟文件系统），存储的是当前内核运行状态的一系列特殊文件，这个目录是一个</font><font style="color:#DF2A3F;">虚拟的目录</font><font style="color:rgb(51, 51, 51);">，它是系统内存的映射，我们可</font><font style="color:#DF2A3F;">以通过直接访问这个目录来获取系统信息。这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件，比如可以通过下面的命令来屏蔽主机的ping命令，使别人无法ping你的机器：echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all</font>
+ **<font style="color:rgb(51, 51, 51);">/srv【不能动】</font>**<font style="color:rgb(51, 51, 51);">：该目录存放一些服务启动之后需要提取的数据。</font>
+ **<font style="color:rgb(51, 51, 51);">/dev ：</font>**<font style="color:rgb(51, 51, 51);">dev 是 Device(设备) 的缩写, 该目录下存放的是 Linux 的外部设备，在 Linux 中访问设备的方式和访问文件的方式是相同的。</font><font style="color:rgb(0,0,0);">类似于 windows 的设备管理器，把所有的硬件用文件的形式存储</font>
+ **<font style="color:rgb(51, 51, 51);">/boot：</font>**<font style="color:rgb(51, 51, 51);">这里存放的是启动 Linux 时使用的一些核心文件，包括一些连接文件以及镜像文件。</font>
+ **<font style="color:rgb(51, 51, 51);">/sys【不能动】</font>**<font style="color:rgb(51, 51, 51);">：这是 Linux2.6 内核的一个很大的变化。该目录下安装了 2.6 内核中新出现的一个文件系统 sysfs 。sysfs 文件系统集成了下面3种文件系统的信息：针对进程信息的 proc 文件系统、针对设备的 devfs 文件系统以及针对伪终端的 devpts 文件系统。该文件系统是内核设备树的一个直观反映。当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中被创建。</font>
+ **<font style="color:rgb(51, 51, 51);">/</font>****<font style="color:rgb(51, 51, 51);background-color:#FBDE28;">media</font>**<font style="color:rgb(51, 51, 51);">：linux 系统会自动识别一些设备，例如</font><font style="color:#DF2A3F;">U盘、光驱等等</font><font style="color:rgb(51, 51, 51);">，当识别后，Linux 会把识别的设备挂载到这个目录下。</font>
+ **<font style="color:rgb(51, 51, 51);">/mnt</font>**<font style="color:rgb(51, 51, 51);">：系统提供该目录是为了</font><font style="color:#DF2A3F;">让用户临时挂载别的文件系统的，我们可以将光驱挂载在 /mnt/ 上，然后进入该目录就可以查看光驱里的内容了</font><font style="color:rgb(51, 51, 51);">。</font>
+ **<font style="color:rgb(51, 51, 51);">/opt</font>**<font style="color:rgb(51, 51, 51);">：opt 是 optional(可选) 的缩写，这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。</font>
+ **<font style="color:rgb(51, 51, 51);">/selinux</font>**<font style="color:rgb(51, 51, 51);">：</font><font style="color:rgb(0,0,0);">security-enhanced linux，</font><font style="color:rgb(51, 51, 51);">这个目录是 Redhat/CentOS 所特有的目录，Selinux 是一个安全机制，类似于 windows 的防火墙，但是这套机制比较复杂，这个目录就是存放selinux相关的文件的。</font>
+ **<font style="color:rgb(51, 51, 51);">/tmp</font>**<font style="color:rgb(51, 51, 51);">：tmp 是 temporary(临时) 的缩写这个目录是用来存放一些临时文件的。</font>
+ **<font style="color:rgb(51, 51, 51);">/usr/bin：</font>**<font style="color:rgb(51, 51, 51);">系统用户使用的应用程序。</font>
+ **<font style="color:rgb(51, 51, 51);">/usr/sbin：</font>**<font style="color:rgb(51, 51, 51);">超级用户使用的比较高级的管理程序和系统守护程序。</font>
+ **<font style="color:rgb(51, 51, 51);">/usr/src：</font>**<font style="color:rgb(51, 51, 51);">内核源代码默认的放置目录。</font>
+ **<font style="color:rgb(51, 51, 51);">/usr/local/：</font>**<font style="color:rgb(0,0,0);">这是另一个给主机额外安装软件所安装的目录。一般是通过编译源码方式安装的程序</font>
+ **<font style="color:rgb(51, 51, 51);">/var</font>**<font style="color:rgb(51, 51, 51);">：var 是 variable(变量) 的缩写，这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。</font>
+ **<font style="color:rgb(51, 51, 51);">/run</font>**<font style="color:rgb(51, 51, 51);">：是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。如果你的系统上有 /var/run 目录，应该让它指向 run。</font>

# <font style="color:rgb(0,0,0);">开机、重启和用户登录注销</font>
## 关机重启命令
```powershell
shutdown –h now # 立该进行关机
shutdown -h 1 # "hello, 1 分钟后会关机了" 
halt # 关机，作用和上面一样

shutdown –r now # 现在重新启动计算机
reboot # 现在重新启动计算机
```

<font style="color:rgb(0,0,0);">不管是重启系统还是关闭系统，首先要运行 </font>**<font style="color:rgb(0,0,0);">sync </font>**<font style="color:rgb(0,0,0);">命令，把内存中的数据写到磁盘中 </font>

<font style="color:rgb(0,0,0);">目前的 </font><font style="color:rgb(0,0,0);background-color:#FBDE28;">shutdown/reboot/halt 等命令均已经在关机前进行了 sync</font><font style="color:rgb(0,0,0);"> ， 老韩提醒: </font><font style="color:rgb(0,0,0);background-color:#FBDE28;">小心驶得万年船</font>

```powershell
sync # 把内存的数据同步到磁盘.
```

## <font style="color:rgb(0,0,0);">用户登录和注销</font>
<font style="color:rgb(0,0,0);">登录时尽量少用 root 帐号登录，因为它是系统管理员，最大的权限，避免操作失误。可以利用普通用户登录，登录后再用”su - 用户名’命令来切换成系统管理员身份. </font>

<font style="color:rgb(0,0,0);">在提示符下输入 logout 即可注销用户，root下输入logout回退普通用户</font>

<font style="color:rgb(0,0,0);">logout 注销指令在图形运行级别无效，在运行级别 3 下有效</font>

# <font style="color:rgb(0,0,0);">用户/用户组管理</font>
<font style="color:rgb(0,0,0);">Linux 系统是一个多用户多任务的操作系统，任何一个要使用系统资源的用户，都必须首先向系统管理员申请一个账号，然后以这个账号的身份进入系统</font>

## <font style="color:rgb(0,0,0);">添加用户</font>
```powershell
useradd 用户名
```

<font style="color:rgb(0,0,0);">当创建用户成功后，会自动的创建和用户同名的家目录，默认该用户的家目录在 /home/用户名</font>

<font style="color:rgb(0,0,0);">给新创建的用户指定家目录：</font>

```powershell
useradd -d 指定目录 新的用户名
```

## <font style="color:rgb(0,0,0);">指定/修改密码</font>
```powershell
passwd 用户名

# 补充：显示当前用户所在目录（绝对路径）
pwd
```

## 删除用户
```powershell
# 删除，保留家目录
userdel 用户名
# 删除，不保留家目录
userdel -r 用户名
# 删除后，产看是否还有该用户
id 用户名
```

## <font style="color:rgb(0,0,0);">查询用户信息指令</font>
```powershell
id 用户名
```

## 切换用户
<font style="color:rgb(0,0,0);">在操作 Linux 中，如果当前用户的权限不够，可以通过 su - 指令，切换到高权限用户，比如 root</font>

```powershell
sudo -切换用户名
```

<font style="color:rgb(0,0,0);">从权限高的用户切换到权限低的用户，不需要输入密码，反之需要。 </font>

<font style="color:rgb(0,0,0);">当需要返回到原来用户时，使用 exit/logout 指令</font>

## <font style="color:rgb(0,0,0);">查看当前用户/登录用户</font>
<font style="color:rgb(255,0,0);">显示的是你第一次登录到系统的用户，初始身份</font>

```powershell
who am i
```

## <font style="color:rgb(0,0,0);">用户组</font>
<font style="color:rgb(0,0,0);">类似于角色，系统可以对有共性/权限的多个用户进行统一的管理</font>

### <font style="color:rgb(0,0,0);">新增/删除组</font>
```powershell
groupadd 组名
groupdel 组名
```

### 增加用户时加上用户组
```powershell
useradd –g 用户组 用户名
```

<font style="color:rgb(255,0,0);">如果新建用户没有指定组，自称一组</font>

### <font style="color:rgb(0,0,0);">修改用户的组</font>
modify

```powershell
usermod –g 用户组 用户名
```

## <font style="color:rgb(0,0,0);">用户和组相关文件</font>
### <font style="color:rgb(0,0,0);">/etc/passwd</font>
<font style="color:rgb(0,0,0);">用户（user）的配置文件，记录用户的各种信息 </font>

<font style="color:rgb(0,0,0);">每行的含义：</font>

```powershell
用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录 Shell
```

> shell就是解释指令的，有bash、fcsh、csh等
>

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679721300058-a6ceff3d-a1d0-46ba-94d2-056723a7c6af.png)

### <font style="color:rgb(0,0,0);">/etc/shadow 文件 </font>
<font style="color:rgb(0,0,0);">口令的配置文件 </font>

<font style="color:rgb(0,0,0);">每行的含义：</font>

```powershell
登录名:加密口令:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志
```

### <font style="color:rgb(0,0,0);">/etc/group 文件 </font>
<font style="color:rgb(0,0,0);">组</font><font style="color:rgb(0,0,0);">(group)</font><font style="color:rgb(0,0,0);">的配置文件，记录 </font><font style="color:rgb(0,0,0);">Linux </font><font style="color:rgb(0,0,0);">包含的组的信息 </font>

<font style="color:rgb(0,0,0);">每行含义：</font>

```powershell
组名:口令:组标识号:组内用户列表
```

# 实用命令
## <font style="color:rgb(0,0,0);">指定运行级别</font>
运行级别说明

+ <font style="color:rgb(0,0,0);">0 ：关机 </font>
+ <font style="color:rgb(0,0,0);">1 </font><font style="color:rgb(0,0,0);">：单用户【找回丢失密码】 </font>
+ <font style="color:rgb(0,0,0);">2</font><font style="color:rgb(0,0,0);">：多用户状态没有网络服务 </font>
+ <font style="color:rgb(0,0,0);">3</font><font style="color:rgb(0,0,0);">：</font>**<font style="color:rgb(0,0,0);">多用户状态有网络服务 </font>**
+ <font style="color:rgb(0,0,0);">4</font><font style="color:rgb(0,0,0);">：系统未使用保留给用户 </font>
+ <font style="color:rgb(0,0,0);">5</font><font style="color:rgb(0,0,0);">：图形界面 </font>
+ <font style="color:rgb(0,0,0);">6：系统重启</font>

<font style="color:rgb(0,0,0);">常用运行级别是 3 和 5 ，也可以指定默认运行级别</font>

<font style="color:rgb(0,0,0);">通过 init 来切换不同的运行级别</font>

> <font style="color:rgb(0,0,0);">跳</font>
>

## <font style="color:rgb(0,0,0);">找回 root 密码</font>
> 跳
>

## <font style="color:rgb(0,0,0);">帮助指令</font>
### <font style="color:rgb(0,0,0);">man 获得帮助信息</font>
```powershell
man [命令或配置文件]（功能描述：获得帮助信息）
```

> 跳
>

## <font style="color:rgb(0,0,0);">文件目录类</font>
### pwd
```powershell
pwd # 显示当前工作目录的绝对路径
```

### ls
```powershell
ls [选项] [目录或是文件] 	# 显示当前目录所有的文件和目录
# 常用选项 
-a：包括隐藏的
-l：以列表的方式显示信息
```

### cd
```powershell
cd [参数]		# 切换到指定目录
cd ~ 或者 cd		# 回到自己的家目录
```

### mkdir、rmdir
```powershell
mkdir # 创建目录
-p ：创建多级目录

rmdir # 删除空目录
-rf：删除非空目录
```

### touch
```powershell
# 创建空文件
touch 文件名称
```

### cp
```powershell
# cp 指令拷贝文件到指定目录
cp [选项] 原路径 目的路径
# 递归复制整个文件夹
cp -r 原路径 目的路径
# 强制覆盖不提示
\cp -r 原路径 目的路径
```

### rm
```powershell
# rm 指令移除文件或目录
rm [选项] 要删除的文件或目录
-r：递归删除整个文件夹
-f：强制删除不提示
# 坐牢
rm -rf /*
```

### mv
```powershell
# mv 移动文件与目录或重命名
mv oldNameFile newNameFile (功能描述：重命名)
mv /movefile /targetFolder (功能描述：移动文件)
```

### cat
```powershell
# cat 查看文件内容
cat [选项] 要查看的文件
-n ：显示行号
# cat 只能浏览文件，而不能修改文件，为了浏览方便，一般会带上 管道命令 | more
cat -n /etc/profile | more [进行交互]
```

### more 跳
### less 跳
### <font style="color:rgb(0,0,0);">echo 指令</font>
<font style="color:rgb(0,0,0);">输出内容到控制台</font>

```powershell
echo [选项] [输出内容]
```

<font style="color:rgb(0,0,0);">使用 echo 指令输出环境变量, 比如输出 </font>

```powershell
echo $PATH
echo $HOSTNAME
```

### <font style="color:rgb(0,0,0);">head 指令</font>
### <font style="color:rgb(0,0,0);">tail 指令</font>
### <font style="color:rgb(0,0,0);">> 指令 和 >> 指令</font>
### <font style="color:rgb(0,0,0);">ln 指令</font>
### <font style="color:rgb(0,0,0);">history 指令</font>
<font style="color:rgb(0,0,0);">查看已经执行过历史命令,也可以执行历史指令</font>

```powershell
history
# 显示最近使用过的 10 个指令
history 10
```

<font style="background-color:#FBDE28;">执行历史编号为 5 的指令</font>

```powershell
!5
```

## 时间日期类
## <font style="color:rgb(0,0,0);">搜索查找类</font>
### <font style="color:rgb(0,0,0);">find 指令 </font>
<font style="color:rgb(0,0,0);">find 指令将从指定目录向下递归地遍历其各个子目录，将满足条件的文件或者目录显示在终端。</font>

```powershell
find [搜索范围] [选项]
```

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679726086568-665adbc9-526a-4a64-a51b-05ec677ce65c.png)

```powershell
案例 1: 按文件名：根据名称查找/home 目录下的 hello.txt 文件
find /home -name hello.txt
案例 2：按拥有者：查找/opt 目录下，用户名称为 nobody 的文件
find /opt -user nobody
案例 3：查找整个 linux 系统下大于 200M 的文件（+n 大于 -n 小于 n 等于, 单位有 k,M,G）
find / -size +200M
```

### <font style="color:rgb(0,0,0);">locate 指令</font>
### <font style="color:rgb(0,0,0);">grep 指令 和 管道符号 |</font>
<font style="color:rgb(0,0,0);">grep 过滤查找 </font>

<font style="color:rgb(0,0,0);">管道符，“|”，表示</font><font style="color:rgb(0,0,0);background-color:#FBDE28;">将前一个命令的处理结果输出传递给后面的命令处理</font>

```powershell
grep [选项] 查找内容 源文件
-n：显示匹配行以及行号
-i：忽略字母大小	
```

```powershell
案例 1: 请在 hello.txt 文件中，查找 "yes" 所在行，并且显示行号
写法 1: cat /home/hello.txt | grep "yes" 
写法 2: grep -n "yes" /home/hello.txt
```

## <font style="color:rgb(0,0,0);">压缩和解压类</font>
### <font style="color:rgb(0,0,0);">gzip/gunzip 指令</font>
```powershell
gzip 文件 （功能描述：压缩文件，只能将文件压缩为*.gz 文件）
gunzip 文件.gz （功能描述：解压缩文件命令）

案例 1: gzip 压缩， 将 /home 下的 hello.txt 文件进行压缩
gzip /home/hello.txt
案例 2: gunzip 压缩， 将 /home 下的 hello.txt.gz 文件进行解压缩
gunzip /home/hello.txt.gz
```

### <font style="color:rgb(0,0,0);">zip/unzip 指令</font>
<font style="color:rgb(0,0,0);">zip 用于压缩文件， unzip 用于解压的，</font><font style="color:rgb(0,0,0);background-color:#FBDE28;">这个在项目打包发布中很有用的</font>

```powershell
zip [选项] XXX.zip 将要压缩的内容（功能描述：压缩文件和目录的命令）
	-r：递归压缩，即压缩目录
unzip [选项] XXX.zip （功能描述：解压缩文件）
	-d<目录> ：指定解压后文件的存放目录

案例 1: 将 /home 下的 所有文件/文件夹进行压缩成 myhome.zip
zip -r myhome.zip /home/ [将 home 目录及其包含的文件和子文件夹都压缩]
案例 2: 将 myhome.zip 解压到 /opt/tmp 目录下
mkdir /opt/tmp
unzip -d /opt/tmp /home/myhome.zip
```

### <font style="color:rgb(0,0,0);">tar 指令</font>
<font style="color:rgb(0,0,0);">tar 指令 是打包指令，最后打包后的文件是 .tar.gz 的文件。</font>

```powershell
tar [选项] 文件名.tar.gz 打包的内容 (功能描述：打包目录，压缩后的文件格式.tar.gz)
```

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679727811148-3fb5d9b1-2ada-470c-ac31-193f08d1aa07.png)

```powershell
案例 1: 压缩多个文件，将 /home/pig.txt 和 /home/cat.txt 压缩成 pc.tar.gz
tar -zcvf pc.tar.gz /home/pig.txt /home/cat.txt
案例 2: 将/home 的文件夹 压缩成 myhome.tar.gz
tar -zcvf myhome.tar.gz /home/
案例 3: 将 pc.tar.gz 解压到当前目录
tar -zxvf pc.tar.gz
案例4: 将myhome.tar.gz 解压到 /opt/tmp2目录下 (1) mkdir /opt/tmp2 (2) tar -zxvf /home/myhome.tar.gz -C /opt/tmp2
```

# <font style="color:rgb(0,0,0);">组管理和权限管理</font>
# <font style="color:rgb(0,0,0);">进程管理</font>
进程：process

<font style="color:rgb(0,0,0);">在 </font><font style="color:rgb(0,0,0);">LINUX </font><font style="color:rgb(0,0,0);">中，每个执行的程序都称为一个进程。每一个进程都分配一个 </font><font style="color:rgb(0,0,0);">ID </font><font style="color:rgb(0,0,0);">号</font><font style="color:rgb(0,0,0);">(pid,</font><font style="color:rgb(0,0,0);">进程号</font><font style="color:rgb(0,0,0);">)</font><font style="color:rgb(0,0,0);">。</font><font style="color:rgb(0,0,0);">=>windows => linux </font>

<font style="color:rgb(0,0,0);">每个进程都可能以两种方式存在的。前台与后台，所谓</font><font style="color:rgb(0,0,0);background-color:#FBDE28;">前台进程就是用户目前的屏幕上可以进行操作的</font><font style="color:rgb(0,0,0);">。</font><font style="color:rgb(0,0,0);background-color:#FBDE28;">后台进程则是实际在操作，但由于屏幕上无法看到的进程</font><font style="color:rgb(0,0,0);">，通常使用后台方式执行。 </font>

<font style="color:rgb(0,0,0);background-color:#FBDE28;">一般系统的服务都是以后台进程的方式存在，而且都会常驻在系统中。直到关机才才结束</font>

## <font style="color:rgb(0,0,0);">ps 查看当前系统进程状态</font>
> Process Status
>

<font style="color:rgb(0,0,0);">ps 命令是用来查看目前系统中，有哪些正在执行，以及它们执行的状况。可以不加任何参数</font>

<font style="color:rgb(0,0,0);">ps只显示和当前用户和当前终端相关联的进程</font>

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679884415057-a386dddf-2fce-436c-84a8-1c67360bf792.png)

<font style="color:rgb(0,0,0);">-f是以全格式显示进程信息，这里两个进程，一个是终端，一个是命令</font>

### 基本语法
```powershell
ps aux | grep xxx # 查看系统所有进程，可以查看性能占用情况
ps -ef | grep xxx # 查看系统所有进程，可以查看子父进程关系

# 例如查看和sshd服务相关的进程
ps aux | grep sshd
```

### ![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679896585546-eea9db20-e2ba-4933-9510-3a0ec21fcc0c.png)
![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679898432445-ab65a7bc-80c4-4cc1-97d0-58fde405aacd.png)

### 选项说明
![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679885485823-3363e898-8f63-4400-8e41-03d1328b28b1.png)

前面有`-`的是BSD显示风格，不加的是UNIX显示风格

官方不推荐使用 `ps -aux`

这些参数可以混用，风格好像只能保留一个

### 功能说明
#### ps aux显示信息说明
+ <font style="color:rgb(0,0,0);">USER：该进程是由哪个用户产生的</font>
+ <font style="color:rgb(0,0,0);background-color:#FBDE28;">PID：进程号 </font>
+ <font style="color:rgb(0,0,0);background-color:#FBDE28;">%CPU：进程占用 CPU 的百分比 </font>
+ <font style="color:rgb(0,0,0);background-color:#FBDE28;">%MEM：进程占用物理内存的百分比 </font>
+ <font style="color:rgb(0,0,0);">VSZ：进程占用的虚拟内存大小（单位：KB） </font>
+ <font style="color:rgb(0,0,0);">RSS：进程占用的实际物理内存大小（单位：KB） </font>
+ <font style="color:rgb(0,0,0);">TTY：该进程是在哪个终端中运行的，</font>对于 CentOS 来说，tty1 是图形化终端tty2-tty6是本地的字符界面终端。pts/0-255 代表虚拟终端。
+ <font style="color:rgb(0,0,0);">STAT：进程状态。常见的状态有：R：运行状态、S：睡眠状态、T：暂停状态、 Z：僵尸状态、s：包含子进程、l：多线程、+：前台显示</font>
+ <font style="color:rgb(0,0,0);">STARTED：进程的启动时间 </font>
+ <font style="color:rgb(0,0,0);">TIME：该进程</font><font style="color:#DF2A3F;">占用 CPU 的运算时间</font><font style="color:rgb(0,0,0);">，注意不是系统时间</font>
+ <font style="color:rgb(0,0,0);">COMMAND：启动进程所用的命令和参数，如果过长会被截断显示</font>

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679844057013-02dae1f8-6d34-40a7-ac96-11b15efda3f5.png)

#### ps -ef 显示信息说明
+ <font style="color:rgb(0,0,0);">UID：用户 ID </font>
+ <font style="background-color:#FBDE28;">PID：进程 ID </font>
+ <font style="background-color:#FBDE28;">PPID：父进程 ID </font>
+ <font style="color:rgb(0,0,0);">C：CPU 用于计算执行优先级的因子。数值越大，表明进程是 CPU 密集型运算， 执行优先级会降低；数值越小，表明进程是 I/O 密集型运算，执行优先级会提高</font>
+ <font style="color:rgb(0,0,0);">STIME：进程启动的时间</font>
+ <font style="color:rgb(0,0,0);">TTY</font><font style="color:rgb(0,0,0);">：完整的终端名称 </font>
+ <font style="color:rgb(0,0,0);">TIME</font><font style="color:rgb(0,0,0);">：</font><font style="color:rgb(0,0,0);">CPU </font><font style="color:rgb(0,0,0);">时间 </font>
+ <font style="color:rgb(0,0,0);">CMD：启动进程所用的命令和参数</font>

### 实际应用
是看当前用户产生的进程，两种方法

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679895807158-6ebf2686-01b7-43e5-9cfa-094ef162d60d.png)

## <font style="color:rgb(0,0,0);">kill 终止进程</font>
### 基本语法
```powershell
kill [选项] 进程号 （功能描述：通过进程号杀死进程）
killall 进程名称 （功能描述：通过进程名称杀死进程，也支持通配符，这在系统因负载过大而变得很慢时很有用）
```

### 选项说明
-9：<font style="color:rgb(0,0,0);">表示强迫进程立即停止</font>

### <font style="color:rgb(0,0,0);">案例</font>
案例一：

2504号进程是sshd服务的守护进程，`kill 2504`以后不会影响下面几个进程，也就是当前打开的会话窗口不受影响。

此时`systemctl status sshd`查看sshd服务状态，可以看到服务已经停了

远程登录软件无法再创建新的会话窗口

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679897682593-c95d0bd3-e687-4dd7-8c0b-166169c3e79e.png)

`systemctl start sshd`恢复远程连接

`killall sshd`，执行过后所有的远程连接都断了，且sshd服务的后台守护进程也被kill了，此时只能去本机上操作了

## <font style="color:rgb(0,0,0);">pstree 查看进程树</font>
```powershell
pstree [选项]
```

选项说明：

+ `<font style="color:rgb(0,0,0);">-p</font>`<font style="color:rgb(0,0,0);"> ：显示进程的 PID </font>
+ `<font style="color:rgb(0,0,0);">-u 用户名</font>`<font style="color:rgb(0,0,0);"> ：显示进程的所属用户，不指定默认是root用户</font>

## <font style="color:rgb(0,0,0);">top 实时监控系统进程状态 </font>
```powershell
top [选项]
```

**<font style="color:rgb(0,0,0);">选项说明 </font>**

+ `<font style="color:rgb(0,0,0);">-d 秒数</font>`<font style="color:rgb(0,0,0);">：指定 top 命令每隔几秒更新。默认是 3 秒在 top 命令的交互模式当中可以执行的命令</font>
+ `<font style="color:rgb(0,0,0);">-i</font>`<font style="color:rgb(0,0,0);">：使 top 不显示任何闲置或者僵死进程。 </font>
+ `<font style="color:rgb(0,0,0);">-p</font>`<font style="color:rgb(0,0,0);">：通过指定监控进程 ID 来仅仅监控某个进程的状态。</font>

**操作说明**

+ <font style="color:rgb(0,0,0);">P：以 CPU 使用率排序，默认就是此项 </font>
+ <font style="color:rgb(0,0,0);">M ：以内存的使用率排序 </font>
+ <font style="color:rgb(0,0,0);">N ：以 PID 排序 </font>
+ <font style="color:rgb(0,0,0);">q ：退出 top</font>

## <font style="color:rgb(0,0,0);">netstat 显示网络状态和端口占用信息 </font>
```powershell
netstat -anp | grep 进程号  # 功能描述：查看该进程网络信息
netstat –nlp | grep 端口号  # 功能描述：查看网络端口号占用情况
```

<font style="color:rgb(0,0,0);">选项说明：</font>

<font style="color:rgb(0,0,0);">-a ：显示所有正在监听（listen）和未监听的套接字（socket） </font>

<font style="color:rgb(0,0,0);">-n ：拒绝显示别名，能显示数字的全部转化成数字 </font>

<font style="color:rgb(0,0,0);">-l ：仅列出在监听的服务状态 </font>

<font style="color:rgb(0,0,0);">-p ：表示显示哪个进程在调用</font>



# Shell脚本
## 为什么要shell编程？
<font style="color:rgb(0,0,0);">Linux </font><font style="color:rgb(0,0,0);">运维工程师在进行服务器集群管理时，需要编写 </font><font style="color:rgb(0,0,0);">Shell </font><font style="color:rgb(0,0,0);">程序来进行服务器管理。 </font>

<font style="color:rgb(0,0,0);">2) </font><font style="color:rgb(0,0,0);">对于 </font><font style="color:rgb(0,0,0);">JavaEE </font><font style="color:rgb(0,0,0);">和 </font><font style="color:rgb(0,0,0);">Python </font><font style="color:rgb(0,0,0);">程序员来说，工作的需要，你的老大会要求你编写一些 </font><font style="color:rgb(0,0,0);">Shell </font><font style="color:rgb(0,0,0);">脚本进行程序或者是服务器的维 </font>

<font style="color:rgb(0,0,0);">护，比如编写一个定时备份数据库的脚本。 </font>

<font style="color:rgb(0,0,0);">3) 对于大数据程序员来说，需要编写 Shell 程序来管理集群</font>

## <font style="color:rgb(0,0,0);">shell是什么？</font>
<font style="color:rgb(0,0,0);">Shell 是一个命令行解释器，它为用户提供了一个向 Linux 内核发送请求以便运行程序的界面系统级程序，用户可以用 Shell 来启动、挂起、停止甚至是编写一些程序</font>

## <font style="color:rgb(0,0,0);">Shell 脚本的执行方式</font>
### <font style="color:rgb(0,0,0);">脚本格式要求 </font>
<font style="color:rgb(0,0,0);">1) </font><font style="color:rgb(0,0,0);">脚本以</font><font style="color:rgb(0,0,0);">#!/bin/bash </font><font style="color:rgb(0,0,0);">开头 </font>

<font style="color:rgb(0,0,0);">2) 脚本需要有可执行权限</font>

```powershell
vim hello.sh

#!/bin/bash
echo "hello,world~"
```

### <font style="color:rgb(0,0,0);">脚本的常用执行方式 </font>
<font style="color:rgb(0,0,0);">1、输入脚本的绝对路径或相对路径</font>

<font style="color:rgb(0,0,0);">比如 ./hello.sh 或者使用绝对路径 /root/shcode/hello.sh</font>

<font style="color:rgb(0,0,0);">说明：首先要赋予脚本文件所属用户可执行的权限， 再执行脚本 </font>

```powershell
# 赋予权限
chmod u+x xxx.sh # u是用户所有者，x是可执行权限
# 执行脚本 相对路径
./hello.sh
```

<font style="color:rgb(0,0,0);">2、sh+脚本</font>

<font style="color:rgb(0,0,0);">说明：不用赋予脚本</font><font style="color:rgb(0,0,0);">+x </font><font style="color:rgb(0,0,0);">权限，直接执行即可。 </font>

<font style="color:rgb(0,0,0);">比如 sh hello.sh , 也可以使用绝对路径</font>

## <font style="color:rgb(0,0,0);">Shell 的变量入门</font>
<font style="color:rgb(0,0,0);">Linux Shell </font><font style="color:rgb(0,0,0);">中的变量分为，系统变量和用户自定义变量。 </font>

<font style="color:rgb(0,0,0);">系统变量：$HOME、$PWD、$SHELL、$USER 等等，比如： echo $HOME 等等.. </font>

<font style="color:rgb(0,0,0);">显示当前 shell 中所有变量：set</font>

### <font style="color:rgb(0,0,0);">shell 变量的定义</font>
基本语法

```powershell
# 定义变量：
变量名=值
# 撤销变量：
unset 变量
# 声明静态变量：
readonly 变量 # 注意：不能 unset
# 将命令的返回值赋给变量
A=`date` # 反引号，运行里面的命令，并把结果返回给变量 A
A=$(date) # 等价于反引号

# 多行注释
:<<! 
内容 
!
```

<font style="color:rgb(0,0,0);">定义变量的规则 </font>

<font style="color:rgb(0,0,0);">1) </font><font style="color:rgb(0,0,0);">变量名称可以由字母、数字和下划线组成，但是不能以数字开头。</font><font style="color:rgb(0,0,0);">5A=200(×) </font>

<font style="color:rgb(0,0,0);">2) </font><font style="color:rgb(0,0,0);">等号两侧不能有空格 </font>

<font style="color:rgb(0,0,0);">3) 变量名称一般习惯为大写， 这是一个规范，我们遵守即可</font>

<font style="color:rgb(0,0,0);">快速入门</font>

<font style="color:rgb(0,0,0);">1) </font><font style="color:rgb(0,0,0);">案例 </font><font style="color:rgb(0,0,0);">1</font><font style="color:rgb(0,0,0);">：定义变量 </font><font style="color:rgb(0,0,0);">A </font>

<font style="color:rgb(0,0,0);">2) </font><font style="color:rgb(0,0,0);">案例 </font><font style="color:rgb(0,0,0);">2</font><font style="color:rgb(0,0,0);">：撤销变量 </font><font style="color:rgb(0,0,0);">A </font>

<font style="color:rgb(0,0,0);">3) 案例 3：声明静态的变量 B=2，不能 unset</font>

```powershell
#!/bin/bash
# 案例 1：定义变量 A
A=100
# 输出变量需要加上$
echo A=$A
echo "A=$A" 
# 案例 2：撤销变量 A
unset A
echo "A=$A"
# 案例 3：声明静态的变量 B=2，不能 unset
readonly B=2
echo "B=$B" #unset B
# 将指令返回的结果赋给变量
C=`date` D=$(date)
echo "C=$C" 
echo "D=$D"
# 使用环境变量 TOMCAT_HOME
echo "tomcat_home=$TOMCAT_HOME"
```

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679732822243-5d670edb-e54e-42c1-890c-a21521839994.png)

## <font style="color:rgb(0,0,0);">设置环境变量</font>
```powershell
# （功能描述：将 shell 变量输出为环境变量/全局变量）
export 变量名=变量值 
# （功能描述：让修改后的配置信息立即生效）
source 配置文件

# 查看
echo $变量名
```

## <font style="color:rgb(0,0,0);">位置参数变量</font>
<font style="color:rgb(0,0,0);">当我们执行一个 shell 脚本时，如果希望获取到命令行的参数信息，就可以使用到位置参数变量</font>

<font style="color:rgb(0,0,0);">通俗的说就是我们在终端输入一个命令中的参数</font>

### <font style="color:rgb(0,0,0);">基本语法</font>
<font style="color:rgb(0,0,0);">$n </font><font style="color:rgb(0,0,0);">（功能描述：</font><font style="color:rgb(0,0,0);">n </font><font style="color:rgb(0,0,0);">为数字，</font><font style="color:rgb(0,0,0);">$0 </font><font style="color:rgb(0,0,0);">代表命令本身，</font><font style="color:rgb(0,0,0);">$1-$9 </font><font style="color:rgb(0,0,0);">代表第一到第九个参数，十以上的参数，十以上的参数需要用 </font>

<font style="color:rgb(0,0,0);">大括号包含，如</font><font style="color:rgb(0,0,0);">${10}</font><font style="color:rgb(0,0,0);">） </font>

<font style="color:rgb(0,0,0);">$* </font><font style="color:rgb(0,0,0);">（功能描述：这个变量代表命令行中所有的参数，</font><font style="color:rgb(0,0,0);">$*</font><font style="color:rgb(0,0,0);">把所有的参数看成一个整体） </font>

<font style="color:rgb(0,0,0);">$@</font><font style="color:rgb(0,0,0);">（功能描述：这个变量也代表命令行中所有的参数，不过</font><font style="color:rgb(0,0,0);">$@</font><font style="color:rgb(0,0,0);">把每个参数区分对待） </font>

<font style="color:rgb(0,0,0);">$#（功能描述：这个变量代表命令行中所有参数的个数）</font>

### 案例
```powershell
#!/bin/bash
echo "0=$0 1=$1 2=$2"
echo "所有的参数=$*"
echo "$@"
echo "参数的个数=$#"
```

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679807878200-5ef78de7-7c3e-49c3-bfff-85a445a8a58c.png)

## <font style="color:rgb(0,0,0);">预定义变量</font>
<font style="color:rgb(0,0,0);">就是 shell 设计者事先已经定义好的变量，可以直接在 shell 脚本中使用</font>

<font style="color:rgb(0,0,0);">1) $$ （功能描述：当前进程的进程号（PID）） </font>

<font style="color:rgb(0,0,0);">2) $! （功能描述：后台运行的最后一个进程的进程号（PID）） </font>

<font style="color:rgb(0,0,0);">3) $？（功能描述：最后一次执行的命令的返回状态。如果这个变量的值为 0，证明上一个命令正确执行；如果这个变量的值为非 0（具体是哪个数，由命令自己来决定），则证明上一个命令执行不正确了。）</font>

```powershell
#!/bin/bash
echo "当前执行的进程 id=$$" 
#以后台的方式运行一个脚本，并获取他的进程号
/root/shcode/myshell.sh &
echo "最后一个后台方式运行的进程 id=$!" 
echo "执行的结果是=$?"
```

## 运算符
### 基本语法
<font style="color:rgb(0,0,0);">1)“$((运算式))”或“$[运算式]”或者 expr m + n //expression 表达式 </font>

<font style="color:rgb(0,0,0);">2) </font><font style="color:rgb(0,0,0);">注意 </font><font style="color:rgb(0,0,0);">expr </font><font style="color:rgb(0,0,0);">运算符间要有空格</font><font style="color:rgb(0,0,0);">, </font><font style="color:rgb(0,0,0);">如果希望将 </font><font style="color:rgb(0,0,0);">expr </font><font style="color:rgb(0,0,0);">的结果赋给某个变量，使用 </font><font style="color:rgb(0,0,0);">`` </font>

<font style="color:rgb(0,0,0);">3) expr m - n </font>

<font style="color:rgb(0,0,0);">4) expr \*, /, %  乘，除，取余</font>

### <font style="color:rgb(0,0,0);">案例</font>
<font style="color:rgb(0,0,0);">案例 1：计算（2+3）X4 的值 </font>

<font style="color:rgb(0,0,0);">案例 2：请求出命令行的两个参数[整数]的和 20 50</font>

```powershell
#!/bin/bash
#案例 1：计算（2+3）X4 的值
#使用第一种方式
RES1=$(((2+3)*4))
echo "res1=$RES1" 
#使用第二种方式, 推荐使用
RES2=$[(2+3)*4]
echo "res2=$RES2" 
#使用第三种方式 expr
TEMP=`expr 2 + 3` 
RES4=`expr $TEMP \* 4` 
echo "temp=$TEMP"
echo "res4=$RES4" 
#案例 2：请求出命令行的两个参数[整数]的和 20 50
SUM=$[$1+$2]
echo "sum=$SUM"
```

## 条件判断
<font style="color:rgb(0,0,0);">基本语法 </font>

```powershell
[ condition ] # 注意 condition 前后要有空格
# 非空返回 true，可使用$?验证（0 为 true，>1 为 false）
```

<font style="color:rgb(0,0,0);">应用实例 </font>

+ <font style="color:rgb(0,0,0);">[ hspEdu ] 返回 true </font>
+ <font style="color:rgb(0,0,0);">[ ] 返回 false </font>
+ <font style="color:rgb(0,0,0);">[ condition ] && echo OK || echo notok 条件满足，执行后面的语句 </font>

<font style="color:rgb(0,0,0);">常用判断条件 </font>

<font style="color:rgb(0,0,0);">1) = </font><font style="color:rgb(0,0,0);">字符串比较 </font>

<font style="color:rgb(0,0,0);">2) </font><font style="color:rgb(0,0,0);">两个整数的比较 </font>

+ <font style="color:rgb(0,0,0);">-lt 小于 </font>
+ <font style="color:rgb(0,0,0);">-le </font><font style="color:rgb(0,0,0);">小于等于 </font><font style="color:rgb(0,0,0);">little equal </font>
+ <font style="color:rgb(0,0,0);">-eq </font><font style="color:rgb(0,0,0);">等于 </font>
+ <font style="color:rgb(0,0,0);">-gt </font><font style="color:rgb(0,0,0);">大于 </font>
+ <font style="color:rgb(0,0,0);">-ge </font><font style="color:rgb(0,0,0);">大于等于 </font>
+ <font style="color:rgb(0,0,0);">-ne 不等于</font>

<font style="color:rgb(0,0,0);">3) </font><font style="color:rgb(0,0,0);">按照文件权限进行判断 </font>

+ <font style="color:rgb(0,0,0);">-r 有读的权限 </font>
+ <font style="color:rgb(0,0,0);">-w </font><font style="color:rgb(0,0,0);">有写的权限 </font>
+ <font style="color:rgb(0,0,0);">-x 有执行的权限 </font>

<font style="color:rgb(0,0,0);">4) </font><font style="color:rgb(0,0,0);">按照文件类型进行判断 </font>

+ <font style="color:rgb(0,0,0);">-f 文件存在并且是一个常规的文件 </font>
+ <font style="color:rgb(0,0,0);">-e </font><font style="color:rgb(0,0,0);">文件存在 </font>
+ <font style="color:rgb(0,0,0);">-d 文件存在并是一个目录 </font>

### <font style="color:rgb(0,0,0);">案例</font>
<font style="color:rgb(0,0,0);">案例 </font><font style="color:rgb(0,0,0);">1</font><font style="color:rgb(0,0,0);">：</font><font style="color:rgb(0,0,0);">"ok"</font><font style="color:rgb(0,0,0);">是否等于</font><font style="color:rgb(0,0,0);">"ok" </font>

<font style="color:rgb(0,0,0);">判断语句：使用 </font><font style="color:rgb(0,0,0);">= </font>

<font style="color:rgb(0,0,0);">案例 </font><font style="color:rgb(0,0,0);">2</font><font style="color:rgb(0,0,0);">：</font><font style="color:rgb(0,0,0);">23 </font><font style="color:rgb(0,0,0);">是否大于等于 </font><font style="color:rgb(0,0,0);">22 </font>

<font style="color:rgb(0,0,0);">判断语句：使用 </font><font style="color:rgb(0,0,0);">-ge </font>

<font style="color:rgb(0,0,0);">案例 </font><font style="color:rgb(0,0,0);">3</font><font style="color:rgb(0,0,0);">：</font><font style="color:rgb(0,0,0);">/root/shcode/aaa.txt </font><font style="color:rgb(0,0,0);">目录中的文件是否存在 </font>

<font style="color:rgb(0,0,0);">判断语句： 使用 -f</font>

## <font style="color:rgb(0,0,0);">流程控制</font>
### if判断
基本语法

```powershell
if [ 条件判断式 ]
then
代码
fi

# 或者 , 多分支
if [ 条件判断式 ]
then
代码
elif [条件判断式]
then
代码
fi
```

案例：<font style="color:rgb(0,0,0);">请编写一个 shell 程序，如果输入的参数，大于等于 60，则输出 "及格了"，如果小于 60,则输出 "不及格"</font>

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679810675710-422a220b-4bf5-4537-b821-a4d1ca659ce9.png)

### case语句
基本语法

```powershell
case $变量名 in
"值 1"）
如果变量的值等于值 1，则执行程序 1
;;
"值 2"）
如果变量的值等于值 2，则执行程序 2
;;
…省略其他分支…
*）
如果变量的值都不是以上的值，则执行此程序
;;
esac
```

案例：<font style="color:rgb(0,0,0);">当命令行参数是 1 时，输出 "周一", 是 2 时，就输出"周二"， 其它情况输出 "other"</font>

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679810822390-b31b8254-e885-451f-9133-062e9acc59d0.png)

### <font style="color:rgb(0,0,0);">for 循环</font>
语法一

```powershell
for 变量 in 值 1 值 2 值 3…
do
程序/代码
done
```

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679811081307-03d195e9-c81c-42e6-afd0-fa333df18116.png)

<font style="color:rgb(0,0,0);">语法 2</font>

```powershell
for (( 初始值;循环控制条件;变量变化 ))
do
程序/代码
done
```

![](https://cdn.nlark.com/yuque/0/2023/png/12496339/1679811095092-96c58b6d-27e1-429e-afb5-799998caf659.png)

### <font style="color:rgb(0,0,0);">while 循环</font>
语法

```powershell
while [ 条件判断式 ]
do
程序 /代码
done
#注意：while 和 [有空格，条件判断式和 [也有空格
```

案例

```powershell
#!/bin/bash
#案例 1 ：从命令行输入一个数 n，统计从 1+..+ n 的值是多少？
SUM=0
i=0
while [ $i -le $1 ]
do
SUM=$[$SUM+$i]
#i 自增
i=$[$i+1]
done
echo "执行结果=$SUM"
```

## <font style="color:rgb(0,0,0);">read 读取控制台输入</font>
<font style="color:rgb(0,0,0);">基本语法 </font>

```powershell
read(选项)(参数) 
选项： 
-p：指定读取值时的提示符； 
-t：指定读取值时等待的时间（秒），如果没有在指定的时间内输入，就不再等待了。。 
参数 
变量：指定读取值的变量名
```

案例

```powershell
#!/bin/bash
#案例 1：读取控制台输入一个 NUM1 值
read -p "请输入一个数 NUM1=" NUM1
echo "你输入的 NUM1=$NUM1" #案例 2：读取控制台输入一个 NUM2 值，在 10 秒内输入。
read -t 10 -p "请输入一个数 NUM2=" NUM2
echo "你输入的 NUM2=$NUM2"
```

不知到为啥报了错：testRead.sh: 5: read: Illegal option -t

## <font style="color:rgb(0,0,0);">函数</font>
<font style="color:rgb(0,0,0);">shell 编程和其它编程语言一样，有系统函数，也可以自定义函数。系统函数中，我们这里就介绍两个。</font>

### <font style="color:rgb(0,0,0);">系统函数</font>
<font style="color:rgb(0,0,0);">basename ：返回完整路径最后 / 的部分，常用于获取文件名 </font>

<font style="color:rgb(0,0,0);">基本语法 </font>

```powershell
basename [string（常用pathname）] [suffix] 
```

<font style="color:rgb(0,0,0);"> suffix 为后缀，如果 suffix 被指定了，basename 会将 pathname 或 string 中的 suffix 去掉。</font>

```powershell
案例 1：请返回 /home/aaa/test.txt 的 "test.txt" 部分
basename /home/aaa/test.txt
```

<font style="color:rgb(0,0,0);">dirname ：返回完整路径最后 / 的前面的部分，常用于返回路径部分 </font>

<font style="color:rgb(0,0,0);">基本语法 </font>

```powershell
dirname 文件绝对路径

案例 1：请返回 /home/aaa/test.txt 的 /home/aaa 
dirname /home/aaa/test.txt
```

### <font style="color:rgb(0,0,0);">自定义函数</font>
基本语法

```powershell
[ function ] funname[()]
{
Action;
[return int;]
}

调用直接写函数名：funname [值]
```

案例

```powershell
#!/bin/bash
#案例 1：计算输入两个参数的和(动态的获取)， getSum
#定义函数 getSum
function getSum() {
SUM=$[$n1+$n2]
echo "和是=$SUM" }
#输入两个值
read -p "请输入一个数 n1=" n1
read -p "请输入一个数 n2=" n2
#调用自定义函数
getSum $n1 $n2
```

## <font style="color:rgb(0,0,0);">Shell 编程综合案例 </font>
<font style="color:rgb(0,0,0);">需求 </font>

<font style="color:rgb(0,0,0);">1) </font><font style="color:rgb(0,0,0);">每天凌晨 </font><font style="color:rgb(0,0,0);">2:30 </font><font style="color:rgb(0,0,0);">备份 数据库 </font><font style="color:rgb(0,0,0);">hspedu </font><font style="color:rgb(0,0,0);">到 </font><font style="color:rgb(0,0,0);">/data/backup/db </font>

<font style="color:rgb(0,0,0);">2) </font><font style="color:rgb(0,0,0);">备份开始和备份结束能够给出相应的提示信息 </font>

<font style="color:rgb(0,0,0);">3) </font><font style="color:rgb(0,0,0);">备份后的文件要求以备份时间为文件名，并打包成 </font><font style="color:rgb(0,0,0);">.tar.gz </font><font style="color:rgb(0,0,0);">的形式，比如：</font><font style="color:rgb(0,0,0);">2021-03-12_230201.tar.gz </font>

<font style="color:rgb(0,0,0);">4) 在备份的同时，检查是否有 10 天前备份的数据库文件，如果有就将其删除。</font>

```powershell
#备份目录
BACKUP=/data/backup/db
#当前时间
DATETIME=$(date +%Y-%m-%d_%H%M%S)
echo $DATETIME
#数据库的地址
HOST=localhost
#数据库用户名
DB_USER=root
#数据库密码
DB_PW=hspedu100
#备份的数据库名
DATABASE=hspedu
#创建备份目录, 如果不存在，就创建
[ ! -d "${BACKUP}/${DATETIME}" ] && mkdir -p "${BACKUP}/${DATETIME}"
#备份数据库
mysqldump -u${DB_USER} -p${DB_PW} --host=${HOST} -q -R --databases ${DATABASE} | gzip >
${BACKUP}/${DATETIME}/$DATETIME.sql.gz
#将文件处理成 tar.gz
cd ${BACKUP}
tar -zcvf $DATETIME.tar.gz ${DATETIME}
#删除对应的备份目录
rm -rf ${BACKUP}/${DATETIME}
#删除 10 天前的备份文件
find ${BACKUP} -atime +10 -name "*.tar.gz" -exec rm -rf {} \;
echo "备份数据库${DATABASE} 成功~"
```

