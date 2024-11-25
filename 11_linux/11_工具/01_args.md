## xargs

`xargs`命令是把接收到的数据重新格式化，再将其作为参数提供给其他命令。

##### `xargs`命令的常用选项包括：

- `-I`：用于指定替换字符串，将输入数据中的特定字符串替换为命令行参数。
- `-n`：用于指定每次执行命令的参数个数。
- `-t`：用于打印执行的命令。
- `-p`：用于提示用户确认是否执行命令。
- `-r`：当标准输入为空时，不执行命令。



下面介绍`xargs`命令的各种使用技巧：

1. ##### 将多行输入转换成单行输入：

   ```shell
   [root@lsq files]# echo -e "1 2 3 4 5 \n6 7 8 \n9 10 11 12" > example.txt
   [root@lsq files]# cat example.txt
   1 2 3 4 5 
   6 7 8 
   9 10 11 12
   [root@lsq files]# cat example.txt | xargs
   1 2 3 4 5 6 7 8 9 10 11 12
   ```

   将单行输入转换成多行输出：

   ```shell
   [root@lsq files]# cat example.txt | xargs -n 3
   1 2 3
   4 5 6
   7 8 9
   10 11 12
   ```

   自定义定界符进行转换（默认的定界符是空格）：

   ```shell
   [root@lsq files]# echo "Hello:Hello:Hello:Hello" | xargs -d : -n 2
   Hello Hello
   Hello Hello
   ```

   

2. 在脚本中使用

   ```shell
   [root@lsq files]# cat echo.sh
   #!/bin/bash
   # 当参数传递给echo.sh后，它会将这些参数打印出来，并且以"^-^"作为结尾
   echo $* '^-^'
   
   [root@lsq files]# echo -e "Tom\nHarry\nJerry\nLucy" > args.txt
   
   [root@lsq files]# cat args.txt 
   Tom
   Harry
   Jerry
   Lucy
   
   [root@lsq files]# cat args.txt | xargs bash echo.sh
   Tom Harry Jerry Lucy ^-^
   
   [root@lsq files]# cat args.txt | xargs -n 2 bash echo.sh
   Tom Harry ^-^
   Jerry Lucy ^-^
   ```

   

   在上面的例子中，我们把参数源都放入args.txt文件，但是除了这些参数，我们还需要一些固定不变的参数，比如：

   ```shell
   [root@lsq files]# bash echo.sh  Welcome Tom
   Welcome Tom ^-^
   ```

   在上述命令执行过程中，Tom是变量，其余部分为常量，我们可以从"args.txt"中提取参数，并按照下面的方式提供给命令：

   ```shell
   [root@lsq files]# bash echo.sh  Welcome Tom
   [root@lsq files]# bash echo.sh  Welcome Herry
   [root@lsq files]# bash echo.sh  Welcome Jerry
   [root@lsq files]# bash echo.sh  Welcome Lucy
   ```

   这时我们需要使用`xargs`中`-I`命令：

   ```shell
   [root@lsq files]# cat args.txt  | xargs -I {} bash echo.sh  Welcome {}
   Welcome Tom ^-^
   Welcome Harry ^-^
   Welcome Jerry ^-^
   Welcome Lucy ^-^
   ```

   

3. ##### 结合`find`使用

   `xargs`和`find`是一对非常好的组合，但是，我们通常是以一种错误的方式运用它们的，比如：

   ```shell
   find . -type f -name "*.txt" -print | xargs rm -f
   ```

   这样做是有<font color='red'>危险</font>的，有时会删除不必删除的文件，如果文件名里包含有空格符(' '),则`xargs`很可能认为它们是定界符（例如，`file text.txt`会被`xargs`误认为`file`和`text.txt`）。

   如果我们想把`find`的输出作为`xargs`的输入，就必须将`-print0`与`find`结合使用以字符`null（'\0'）`来分隔输出，用`find`找出所有`.txt`的文件，然后用`xargs`将这些文件删除：

   ```shell
   find . -type f -name "*.txt" -print0 | xargs -0 rm -f
   ```

   这样就可以删除所有的.txt文件了，xargs -0 将\0作为输入定界符；或者MV掉

   ```shell
   find . -type f -name "*.txt" -print0 | xargs -0 -i mv {} /tmp
   ```

   

4. ##### 运用while语句和子shell

   ```shell
   cat files.txt | (while read arg ;do cat $arg;done)
   ```

   这条命令等同于：

   ```shell
   cat files.txt  | xargs -I {} cat {}
   ```

   在while循环中，可以将cat $arg替换成任意数量的命令，这样我们就可以对同一个参数执行多条命令，也可以不借助管道，将输出传递给其他命令，这个技巧适应于多种问题场景。子shell操作符内部的多个命令可作为一个整体来运行。

5. 