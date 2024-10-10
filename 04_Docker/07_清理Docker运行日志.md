在使用 `docker` 时，往往会出现磁盘空间不足，导致该问题的通常原因是因为 `docker` 中部署的系统输出了大量的日志内容。



### 1.进入`docker`目录，并查看目录下文件大小

```shell
cd /var/lib/docker/
du -sh *

#示例
[root@lsq docker]# du -sh *
107M	containers
28M	image
208K	network
40G	overlay2
20K	plugins
4.0K	swarm
4.0K	tmp
4.0K	trust
8.9G	volumes
```



### 2.查找空间占用很高的目录，进入并清理日志文件

```shell
cd /var/lib/docker/containers/06a75071331cab0d47517a321698253ce51bf8310e5bdd7aeae0f7819c785130

#把日志文件置空
cat /dev/null > *-json.log

#示例
[root@lsq containers]# cd 06a75071331cab0d47517a321698253ce51bf8310e5bdd7aeae0f7819c785130/

[root@lsq 06a75071331cab0d47517a321698253ce51bf8310e5bdd7aeae0f7819c785130]# ll
total 240
-r-------- 1 root root   7763 Oct 10 11:15 06a75071331cab0d47517a321698253ce51bf8310e5bdd7aeae0f7819c785130-json.log
drwx------ 2 root root   4096 Nov 29  2023 checkpoints
-rw-r--r-- 1 root root   5119 Oct 10 09:24 config.v2.json
-rw-r--r-- 1 root root   1428 Oct 10 09:24 hostconfig.json
-rw-r--r-- 1 root root     13 Oct 10 09:24 hostname
-rw-r--r-- 1 root root    175 Oct 10 09:24 hosts
-rw-r--r-- 1 root root     88 Oct 10 09:24 resolv.conf
-rw-r--r-- 1 root root     71 Oct 10 09:24 resolv.conf.hash
drwxr-xr-x 3 root root   4096 Oct 10 09:24 secrets
drwxrwxrwt 2 root root     40 Oct 10 09:24 shm

[root@lsq 06a75071331cab0d47517a321698253ce51bf8310e5bdd7aeae0f7819c785130]# cat /dev/null > *-json.log

[root@lsq 06a75071331cab0d47517a321698253ce51bf8310e5bdd7aeae0f7819c785130]# ll
total 232
-r-------- 1 root root      0 Oct 10 13:36 06a75071331cab0d47517a321698253ce51bf8310e5bdd7aeae0f7819c785130-json.log
drwx------ 2 root root   4096 Nov 29  2023 checkpoints
-rw-r--r-- 1 root root   5119 Oct 10 09:24 config.v2.json
-rw-r--r-- 1 root root   1428 Oct 10 09:24 hostconfig.json
-rw-r--r-- 1 root root     13 Oct 10 09:24 hostname
-rw-r--r-- 1 root root    175 Oct 10 09:24 hosts
-rw-r--r-- 1 root root     88 Oct 10 09:24 resolv.conf
-rw-r--r-- 1 root root     71 Oct 10 09:24 resolv.conf.hash
drwxr-xr-x 3 root root   4096 Oct 10 09:24 secrets
drwxrwxrwt 2 root root     40 Oct 10 09:24 shm

```



也可以**过滤**出特定大小的文件进行清理：

```shell
#过滤出大小达到 G 的文件夹名：
`du -sh ./* | grep G | awk '{print $2}'`
```



### 3.快速处理

要查看 Docker 日志的大小，可以运行以下命令：

```shell
du -hs /var/lib/docker/containers/*/*-json.log

#示例
[root@lsq containers]# du -hs /var/lib/docker/containers/*/*-json.log
0	/var/lib/docker/containers/06a75071331cab0d47517a321698253ce51bf8310e5bdd7aeae0f7819c785130/06a75071331cab0d47517a321698253ce51bf8310e5bdd7aeae0f7819c785130-json.log
81M	/var/lib/docker/containers/873de8a3c37de857dde809fb676bcd6b72aceb0bbc899c0177704fe3ad6e8909/873de8a3c37de857dde809fb676bcd6b72aceb0bbc899c0177704fe3ad6e8909-json.log
140K	/var/lib/docker/containers/b6fc46848dbbe344ef092da519a1f039bad3a97ddf18770ca77c97db7ba1c56f/b6fc46848dbbe344ef092da519a1f039bad3a97ddf18770ca77c97db7ba1c56f-json.log
```



这个命令将显示每个容器的日志大小，并将其按递增的顺序列出。

用户可以从中找到 `Docker` 容器日志的大小，并确定是否需要进行操作。



使用以下命令删除不需要的日志文件：

```shell
find /var/lib/docker/containers/ -name "*-json.log" | xargs rm -f
```

