`docker inspect` 会以 json 格式得到 docker 镜像/容器的元数据。

### `-f`

通常我们需要获取某一个具体的key，会用grep，如下：

```bash
# 不完整数据
[root@lsq docker]# docker inspect gateway | grep Networks
            "Networks": {
```



grep会获取到其他的数据，不够完整或者有冗余，还得进一步处理：

```shell
# 完整数据
[root@lsq docker]# docker inspect gateway | grep -A 18 Networks
            "Networks": {
                "docker_beta-network": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": [
                        "8df847808fa0",
                        "gateway"
                    ],
                    "NetworkID": "8e01c010d5207f190444a5b4ddbb437041e6bb5962680f46db74a64e673b2a89",
                    "EndpointID": "1a5d88d977c06a52df67b1c96fcbc9c96b7e79e12a7c98ead8110b8e5ad039dd",
                    "Gateway": "172.19.0.1",
                    "IPAddress": "172.19.0.3",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:13:00:03"
                }
            }
```

但是 -f 可以解决这个问题。

#### 格式要求：`'{{.一级key值.二级key值}}'`

```shell
[root@lsq docker]# docker inspect -f '{{.NetworkSettings.Networks}}' gateway
map[docker_beta-network:0xc420190000]

# 以json格式输出
[root@lsq docker]# docker inspect -f '{{json .NetworkSettings.Networks}}' gateway
{"docker_beta-network":{"IPAMConfig":null,"Links":null,"Aliases":["8df847808fa0","gateway"],"NetworkID":"8e01c010d5207f190444a5b4ddbb437041e6bb5962680f46db74a64e673b2a89","EndpointID":"1a5d88d977c06a52df67b1c96fcbc9c96b7e79e12a7c98ead8110b8e5ad039dd","Gateway":"172.19.0.1","IPAddress":"172.19.0.3","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"02:42:ac:13:00:03"}}
```



### `--type`：指定具体类型 

如：

--type container 声明查看容器的元数据；

--type image 声明查看镜像的元数据；



### `-s`：显示总的文件大小

```shell
[root@lsq docker]# docker inspect -s gateway | grep Size
            "ShmSize": 67108864,
            "ConsoleSize": [
        "SizeRw": 45240,
        "SizeRootFs": 643240587,
[root@lsq docker]# docker inspect gateway | grep Size
            "ShmSize": 67108864,
            "ConsoleSize": [
```



### 3.`docker inspect -f`更多用法

简单地说，-f 的实参是个 Go 模版，并在容器/镜像的元数据上以该 Go 模版作为输入，最终返回模版指定的数据。

Go 模版是一种模板引擎，让数据以指定的模式输出。

#### 3.1Go 模版详解

##### 模板指令

{{ }} 语法用于处理模版指令，大括号外的任何字符都将直接输出。

##### 3.1.1 上下文

“.” 表示“当前上下文”。

大多数情况下表示了容器元数据的整个数据结构，但在某些情况下可以重新规定上下文，比如使用 with 函数：

```shell
[root@lsq docker]# docker inspect -f '{{.State.Pid}}' gateway
30469
[root@lsq docker]# docker inspect -f '{{with .State}}{{.Pid}}{{end}}' gateway
30469
```



##### 3.1.2 $

可以使用 $ 来获取根上下文，只能获取一级key值

```shell
[root@lsq docker]# docker inspect -f '{{$.Name}} has pid {{with .State}}{{.Pid}}{{end}}' gateway
/gateway has pid 30469
```

注意，单独使用 “.” 本身也是可以的，将输出未格式化的完整元数据。



#### 3.2 数据结构

inspect 数据使用 map 以及数组保存。Map 结构可以通过 . 的链式来访问 map 内部数据：

```shell
[root@lsq docker]# docker inspect -f '{{.NetworkSettings.Networks}}' gateway
map[docker_beta-network:0xc42014a000]
```



##### 3.2.1 如果需要获取的属性名称包含 “/”（比如下列示例数据）或者以数字开头，则不能直接通过级联调用获取信息。因为属性名称中的点号会被解析成级联信息，进而导致返回错误结果。即便使用引号将其包含也会提示语法格式错误。此时，需要通过 index 来读取指定属性信息。

前面卷的例子可以这样写：

```shell
# 错误写法
[root@lsq docker]# docker inspect -f '{{.NetworkSettings.Ports.8550/tcp}}' gateway
Template parsing error: template: :1: unexpected ".8550" in operand

# 添加双引号也不行
[root@lsq docker]# docker inspect -f '{{.NetworkSettings.Ports."8550/tcp"}}' gateway
Template parsing error: template: :1: bad character U+0022 '"'

# 正确写法，！！！注意 index 与之后的数据之间包含空格，与之后双引号括起来的也有空格，否则报错
[root@lsq docker]# docker inspect -f '{{index .NetworkSettings.Ports "8550/tcp"}}' gateway
[map[HostIp:0.0.0.0 HostPort:8550]]
```



##### 3.2.2 如果返回结果是一个 map, slice, array 或 string，则可以使用 index 加索引序号（从零开始计数）来读取属性值

```shell
[root@lsq docker]# docker inspect -f '{{.Config.Env}}' gateway
[TZ=Asia/Shanghai PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin LANG=C.UTF-8 JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 JAVA_VERSION=8u111 JAVA_DEBIAN_VERSION=8u111-b14-2~bpo8+1 CA_CERTIFICATES_JAVA_VERSION=20140324]

[root@lsq docker]# docker inspect -f '{{index .Config.Env 0}}' gateway
TZ=Asia/Shanghai
```



#### 3.4 其他常用函数

and

or

eq

ne

lt

le

gt

ge



#### 3.5 range

range 用于遍历结构内返回值的所有数据。支持的类型包括 array, slice, map 和 channel。

使用要点：

1. 对应的值长度为 0 时，range 不会执行。
2.  结构内部如要使用外部的变量，需要在前面加 引用，比如Var2。
3. range 也支持 else 操作。效果是：当返回值为空或长度为 0 时执行 else 内的内容。

```shell
{{range pipeline}}{{.}}{{end}}
{{range pipeline}}{{.}}{{else}}{{.}}{{end}}

* 查看容器网络下已挂载的所有容器名称，如果没有挂载任何容器，则输出 "With No Containers"
[root@lsq docker]# docker inspect --format '{{range .Containers}}{{.Name}}{{println}}{{else}}With No Containers{{end}}' docker_beta-network
gateway
pgsql
```



#### 3.6 docker 内置函数

##### 3.6.1 json

Docker 默认以字符串显示返回结果。而该函数可以将结果格式化为压缩后的 json 格式数据。

```shell
[root@lsq docker]# docker inspect --format='{{json .Config}}' gateway
{"Hostname":"gateway","Domainname":"","User":"","AttachStdin":false,"AttachStdout":false,"AttachStderr":false,"ExposedPorts":{"8550/tcp":{}},"Tty":false,"OpenStdin":false,"StdinOnce":false,"Env":["TZ=Asia/Shanghai","PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin","LANG=C.UTF-8","JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64","JAVA_VERSION=8u111","JAVA_DEBIAN_VERSION=8u111-b14-2~bpo8+1","CA_CERTIFICATES_JAVA_VERSION=20140324"],"Cmd":["/bin/bash","-c","java -jar /mnt/gateway-0.0.1-SNAPSHOT.jar --spring.profiles.active=beta"],"Image":"java:8","Volumes":{"/mnt":{}},"WorkingDir":"","Entrypoint":null,"OnBuild":null,"Labels":{"com.docker.compose.config-hash":"0492072c3408f28a1e7ef88e8d7208288e577cd2f70b7092690d3ea3e35975b2","com.docker.compose.container-number":"1","com.docker.compose.oneoff":"False","com.docker.compose.project":"docker","com.docker.compose.service":"gateway","com.docker.compose.version":"1.18.0"}}
```



##### 3.6.2 join

用指定的字符串将返回结果连接后一起展示。操作对象必须是字符串数组。

```shell
[root@lsq docker]# docker inspect --format '{{join .Config.Env " , "}}' gateway
TZ=Asia/Shanghai , PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin , LANG=C.UTF-8 , JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64 , JAVA_VERSION=8u111 , JAVA_DEBIAN_VERSION=8u111-b14-2~bpo8+1 , CA_CERTIFICATES_JAVA_VERSION=20140324
```



##### 3.6.3 split

使用指定分隔符将返回结果拆分为字符串列表。操作对象必须是字符串且不能是纯数字。同时，字符串中必须包含相应的分隔符，否则会直接忽略操作。

```shell
[root@lsq docker]# docker inspect --format '{{split .HostsPath "/"}}' gateway
[ var lib docker containers 8df847808fa07a0b1e10c05804a7151c57c87a2e686bce2cc80531080ff09450 hosts]
```



#### 3.7 常用`docker inspect --format`输出示例

##### 3.7.1 获取容器的IP (后面使用容器名或容器ID都可以)

```shell
[root@lsq docker]# docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q)
172.19.0.21
172.19.0.9
172.19.0.17
```



##### 3.7.2 获取容器名称

```shell
[root@lsq docker]# docker inspect --format='{{.Name}}' $(docker ps -aq) | cut -d "/" -f 2
dashboard
uptime
certbot
```



##### 3.7.3 Hostname Name IP

```shell
[root@lsq docker]# docker inspect --format 'Hostname:{{ .Config.Hostname }}  Name:{{.Name}} IP:{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q)
Hostname:customize  Name:/customize IP:172.19.0.21
Hostname:dashboard  Name:/dashboard IP:172.19.0.9
Hostname:uptime  Name:/uptime IP:172.19.0.17
```

