## 常用命令有哪些？

- `ls -l`：输出文件列表
- `nohup`：运行长时间运行的任务，用户退出登录也能运行，可以通过 `&` 后台运行
- `ping`：测试网络的连通性
- `ps`：查看进程信息

## 如何确定另一台主机上的进程是否可以通信？

1）使用 `ping` 测试基本连通性

2）检查端口连通性，使用 `telnet` 或 `nc` (NetCat) 测试特定端口是否开放以及能否建立连接

```
telnet <目标主机IP或域名> <端口号> 
nc -zv <目标主机IP或域名> <端口号>
```

3）使用 `nmap` 扫描扫描远程主机开放的端口

```
nmap -A <目标主机IP或域名>
```

4）使用 `ss` 或 `netstat` 查看本地连接

> 如果可以直接操作远程主机

```
ss -tnlp | grep <目标主机IP或端口> 
netstat -anp | grep <目标主机IP或端口>
```

5）检查防火墙规则

```
sudo iptables -L -n 
sudo firewall-cmd --list-all
```

6）对于 HTTP(S) 服务，可以使用 `curl` 或 `wget`


## 怎么查询日志？

[Linux查看log日志命令总结_linux log日志-CSDN博客](https://blog.csdn.net/ZGL_cyy/article/details/128782594)

1）tail 实时查看

`tail -f -n 20 server.log`：实时查看最后 20 行内容

2）cat 配合 `| grep` 关键字搜索

`cat -n filename |grep “http-nio-8091-exec-7”`：cat 区别于 tail 是对日志进行全文搜索，`-n` 显示行号

3）less 整体阅读

4）vim 编辑器整体查看

5）远程下载到本地查看


