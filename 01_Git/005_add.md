1. ### 工作区、暂存区、版本库

   - **工作区**：电脑中能看到的目录，也就是写代码的地方；
   - **暂存区**：英文叫`stage`或`index`，一般存放在`.git`文件夹下的`index`文件中，暂存区有时也叫索引；
   - **版本库**：在工作区中有一个隐藏目录`.git`，这个不算工作区，而是`git`的版本库；

   

2. ### add基本操作

   ##### add命令的作用就是将工作区的文件添加到暂存区：

   1. `# 将某些文件提交到暂存区`

      ```shell
      git add <file1> <file2>
      ```

   2. `# 将某些目录提交到暂存区`

      ```shell
      git add <folder1> <folder2>
      ```

   

3. ### add命令参数

   **-A： `--all add changes from all tracked and untracked files`（添加所有跟踪和未跟踪文件的更改）**

   -A参数会监控工作区的状态树，它会把工作区的所有变化提交到暂存区，包括`修改(modified)`、`新文件(Untracked files)`、`删除的文件(deleted)`。使用`.`在`git 2.x`也可以达到一样的效果，但在`git 1.x`中不同的是`.`不会监控删除的文件。

   在`git 2.x`中，下面两种用法的效果完全相同。

   - `git add .`
   - `git add -A`

   

   **-u:`--update update tracked files`只更新已被跟踪文件，只监控已经被add的文件，也就是`tracked files`，不会监控没有被跟踪的新文件**

   - `git add -u`

   

4. ### git add 背后做了什么

   先说**结论**：`git add`会在`.git/objects`目录下面创建一个目录和文件，并且在`.git/index`文件中添加一行内容。（这里会说到`git cat-file`命令，虽然平时不怎么用，但是它能帮助我们理解`git add`背后到底做了什么）。

   

   ##### 创建一个 git 仓库，用于查看 git add 背后做了什么操作：

   ```shell
   $ git init
   Initialized empty Git repository in D:/MyPrograms/gittest/.git/
   
   $ echo 'hello git' >> 1.txt
   
   $ git add 1.txt
   warning: LF will be replaced by CRLF in 1.txt.
   The file will have its original line endings in your working directory
   ```

   

   ##### 执行 git add 后:

   1. `git`会将工作区中的文件使用`hash sha-1`算法得到`40`位的`blob`对象`hash`字符串文件，文件中存储的是文件类型和使用算法压缩后的内容，如果查看文件的原始内容，需要使用**`git cat-file -p <hash>`**。这个文件存放在`.git/objects`目录下。

      ```shell
      $ cat .git/objects/8d/0e41234f24b6da002d962a26c2495ea16a425f
      xK▒▒OR04`▒H▒▒▒WH▒,▒6A▒
      
      $ git cat-file -p 8d0e41234
      hello git
      ```

      

   2. `git`会在`.git/index`文件中增加一行内容，就是`hash`值对应的文件名。此时就实现了文件名和内容相对应的操作。

      ```shell
      # 查看暂存区中的文件名
      $ git ls-files
      1.txt
      
      # 查看暂存区中的文件更多信息
      $ git ls-files -s
      100644 8d0e41234f24b6da002d962a26c2495ea16a425f 0       1.txt
      
      ```

      **备注**：`100644`指的是文件权限，`hash`字符串对应`.git/objects`目录下的文件。

      

   ##### `# 查看hash文件`

   ```shell
   # 查看文件类型
   $ git cat-file -t 8d0e41234f24b6da002d962a26c2495ea16a425f
   blob
   
   # 查看文件内容
   $ git cat-file -p 8d0e41234f24b6da002d962a26c2495ea16a425f
   hello git
   
   ```

   `文件类型的返回值种类`

   | 类型   | 描述                     |
   | ------ | ------------------------ |
   | blob   | 存储的是工作区文件的内容 |
   | tree   | 工作树                   |
   | commit | 提交记录信息以及工作树   |

   

5. 

6. ### 
