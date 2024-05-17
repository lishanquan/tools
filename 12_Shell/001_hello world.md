### hello world

编写我们的第一个 `Shell` 脚本 `hello.sh`

```bash
#!/bin/bash

# 执行的命令主体
ls
echo "hello world"
```

- `#!/bin/bash` 指定脚本要使用的 `Shell` 类型为 `Bash` 。
- `#!` 被称为 `Sha-bang` ，或者 `Shebang` ， `Linux` 会分析它之后的指令，并载入该指令作为解析器。
- `ls` 就是脚本文件的内容了，表明我们执行 `hello.sh` 脚本时会列举出当前目录的文件列表并且会向控制台打印 `hello world` 。



**注意：**

如果我们不是 `root` 用户的话，需要给脚本添加可执行权限才可以运行， `chmod +x hello.sh` 。

增加执行权限后可以执行 `./hello.sh` 来运行该脚本，也可以使用 `bash -x hello.sh` 以调试模式运行脚本。



#### 系统命令

上面是通过 `./hello.sh` 的方式执行的脚本文件，每次都需要添加路径，那么我们可以像执行 `ls` 一样，直接执行 `hello.sh` 吗，答案是可以的。

1. `echo $PATH` 获取系统里所有可以被直接执行程序的路径。
2. `sudo cp hello.sh /usr/bin` 将 `hello.sh` 拷贝到上诉任意一个 `path` 路径路径中，这里拷贝到 `/usr/bin` 。
3. 现在可以直接运行 `hello.sh` 命令而不需要添加路径了。