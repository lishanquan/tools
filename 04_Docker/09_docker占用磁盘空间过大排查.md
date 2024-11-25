#### 1. 进入到 /var/lib/docker/overlay2 目录下

```shell
cd /var/lib/docker/overlay2
```

#### 2. 查看谁占用容间最大

```shell
[root@webapi overlay2]# du -h -d 1 | grep G | sort -nr
14G	.
2.9G	./52269e2c696e901c0425c52266c9740bebf94a8d79586efdda5824d2fdd81bd6
2.6G	./9a6f9854a72c755d4f105bdc80673421bee9421fe7b81a788729dee8f1d7b039
```

#### 3. 通过目录名查找容器名

```shell
[root@webapi overlay2]# docker ps -q | xargs docker inspect --format '{{.State.Pid}}, {{.Id}}, {{.Name}}, {{.GraphDriver.Data.WorkDir}}' | grep 9a6f9854a72c755d4f105bdc80673421bee9421fe7b81a788729dee8f1d7b039
23557, abfcafd4070bd17c124dbacf8ee19c12ebe37ed57e927a4ab0a0abcaad078741, /member, /var/lib/docker/overlay2/9a6f9854a72c755d4f105bdc80673421bee9421fe7b81a788729dee8f1d7b039/work
```

