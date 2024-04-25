在修改 `nginx`配置文件时，遇到了如下错误：

```shell
nginx: [emerg] duplicate upstream "xxx" in /etc/nginx/conf.d/default.conf:7
```

检查确认配置文件中没有重复的`upstream "xxx"`

最终确认原因为**同目录下的备份配置同样会被nginx加载**：

```shell
[root@iZwz94ke64mjuw9gbsqbs7Z docker]# ll nginx/conf.d/
total 12
-rw-r--r-- 1 root root 11101 Apr 24 17:07 default-bak.conf
-rw-r--r-- 1 root root 11101 Apr 24 17:07 default.conf
```

备份的文件不能以`conf`结尾，可以修改为`default.conf.bak`。