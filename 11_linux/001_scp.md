# SCP

```shell
scp [-r] 参数1 参数2
```

- `-r`选项用于复制文件夹使用，如果复制文件夹，必须使用`-r`
- 参数1：本机路径
- 参数2：远程目标路径



1. ### 常规用法

   将`test`文件夹复制到`node2`对应文件夹`/home/fovace/`

   ```shell
    scp -r ./test node2:/home/fovace/
   ```

   

2. 高级用法

   将`test`文件夹复制到`node2`**同名路径**下

   ```shell
   scp -r ./test node2:`pwd`/
   ```

   简化：

   ```shell
   scp -r ./test node2:$PWD
   ```

   

3. 