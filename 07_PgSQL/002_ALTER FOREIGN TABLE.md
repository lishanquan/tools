1. ### 删除远程分支

   ```shell
   git push origin --delete 分支名     （删除远程分支，并推送）
   ```

   简写：

   ```shell
   git push origin -d 分支名
   ```

   1. #### 问题1：remote ref does not exist。

      原因是远程分支已经被删除，但是我们使用git branch -a 查看的时候，仍然会看到那个分支，这时候就用到下面的命令了：

      ```shell
      # 先清除远程分支的本地缓存
      git fetch -p origin  
      ```

   2. #### 问题2：使用git 管理项目的时候，编译过程中出现了很多中间文件

      ```shell
      # 清除分支上无法提交的文件，删除 untracked files
      git clean -f
      ```

2. ### git clean

   
