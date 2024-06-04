### declare

`declare` 命令可以声明一些特殊类型的变量，为变量设置一些限制，比如声明只读类型的变量和整数类型的变量。

#### 语法格式

```bash
declare OPTION VARIABLE=value
```



#### 常用参数

- `-r` 将变量设为只读；
- `-i` 将变量设为整数；
- `-a` 将变量定义为数组；
- `-f` 显示此脚本前定义过的所有函数及内容；
- `-F` 仅显示此脚本前定义过的函数名；
- `-x` 将变量声明为环境变量。

`declare`命令如果用在函数中，声明的变量只在函数内部有效，等同于`local`命令。

不带任何参数时，`declare`命令输出当前环境的所有变量，包括函数在内，等同于不带有任何参数的`set`命令。



##### `-i`

`-i` 参数声明整数变量以后，可以直接进行数学运算。

```bash
declare -i val1=12 val2=5
declare -i result
result=val1*val2
echo $result # 60
```



##### `-x`

`-x` 参数等同于 `export` 命令，可以输出一个变量为子 `Shell` 的环境变量。

```bash
declare -x foo=3 

# 等同于
export foo=3 
```



##### `-r`

`-r` 参数可以声明只读变量，无法改变变量值，也不能 `unset` 变量。

```bash
declare -r bar=1

# 如果此时更改bar
bar=2 # -bash: bar：只读变量
```



##### `-u` 

`-u` 参数声明变量为大写字母，可以自动把变量值转成大写字母。

```bash
declare -u foo
foo=upper
echo $foo # UPPER
```



##### `-l` 

`-l` 参数声明变量为小写字母，可以自动把变量值转成小写字母。

```bash
declare -l bar
bar=LOWER
echo $bar # lower
```



##### `-p` 

`-p` 参数输出变量信息。

```bash
foo=hello
declare -p foo #控制台输出：declare -x foo="hello"
```