### echo

`echo` 命令的作用是在屏幕输出一行文本，可以将该命令的参数原样输出。

```bash
echo hello world # 输出当行文本

# 输出多行文本
echo "
  hello
  world
"
```



用 `-e` 参数使得 `echo` 可以解析转义字符

```bash
echo -e "hello \n world" # 如果不添加 -e 则会原样输出，添加了 -e 输出则会换行
```

