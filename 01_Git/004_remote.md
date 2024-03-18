1. ### 推送代码流水线

   1. `# 第一步: 创建本地库并完成初始提交，也就是让本地库有提交记录`
   
   2. `git init`
   
   3. `git add .`
   
   4. `git commit -m "first commit"`
   
   5. `# 第二步: 添加一个远程仓库配置`
   
   6. `git remote add origin https://gitee.com/end/test.git`
   
   7. `# 第三步: 设置上游分支，并且使用远程名称推送到远程库`
   
   8. `git push -u origin master`
   
      
   
2. ### 添加远程仓库楝配置

   ##### 首次将代码推送到远程仓库出现以下提示：

   1. `# 没有配置推送目标`
   2. `fatal: No configured push destination.`
   3. `# 从命令行指定 URL，或使用配置远程存储库`
   4. `Either specify the URL from the command-line or configure a remote repository using`
   5. `# 然后使用远程名称推送`
   6. `and then push using the remote name`

   

   ##### 从命令行指定 URL

   1. `# 命令格式`
   2. `git push <url> <branch>`
   3. `# 使用示例`
   4. `git push git@gitee.com:end/test.git master`

   

   ##### 先配置一个远程存储库，然后使用远程名称推送（其实就是给远程库 url 起了一个比较短的名称，然后使用短名称推送）

   1. `# 命令格式`
   2. `git remote add <name> <url>`
   3. `git push <name> <branch>`
   4. `# 使用示例`
   5. `git remote add origin git@gitee.com:end/test.git`
   6. `git push origin master`

   

3. ### 修改远程库配置

   **如果本地仓库已经关联过远程仓库，使用 `git remote add` 直接关联新的远程库时会报错**

   ```shell
   fatal: remote origin already exists.
   ```

   

   ##### 解决方案1：

   **先删除以前的远程仓库关联关系，再关联到新的远程仓库**

   1. `git remote rm origin`
   2. `git remote add origin https://gitee.com/end/test-2.git`

   

   ##### 解决方案2：

   **使用 <font color="red">git remote set-url origin</font> 直接修改关联的远程仓库**

   1. `# 使用前提: 远程名称 origin 已存在`
   2. `git remote set-url origin https://gitee.com/end/test-2.git`

   

   ##### 修改关联的远程仓库<font color="red">总结</font>：

   1. `# 方案1: 先删除远程库配置，再重新添加`
   2. `git remote rm <name>`
   3. `git remote add <name> <url>`
   4. `# 方案2: 使用 set-url 直接修改`
   5. `git remote set-url <name> <url>`

   

4. ### 删除远程库配置

   `# 命令格式`

   ```shell
   git remote remove <name>
   ```

   `# 使用示例`

   ```shell
   git remote remove origin
   ```

   

5. ### 重命名远程库配置

   `# 命令格式`

   ```shell
   git remote rename <old> <new>
   ```

   `# 使用示例`

   ```shell
   git remote rename origin lsq
   ```

   

6. ### 推送到多个仓库

   （**例如：我们要将代码推送到 gitee 和 github 两个平台**）

   

   1. ##### 添加远程库配置

      - `# gitee`

      ```shell
      git remote add gitee git@gitee.com:end/test.git
      ```

      - `# github`

      ```shell
      git remote add github git@github.com:end/test.git
      ```

      

   2. ##### 将代码推送 gitee 和 github

      ```shell
      git push gitee master && git push github master
      ```

      

   3. ##### 推送到远程库时，因为命令有点长，我们可以定义一个系统配置别名，进而简化推送命令

      `# mac 用户可以在 ~/.zshrc 文件中增加以下配置，通过别名 gp 推送`

      ```shell
      alias gp="git push gitee master && git push github master"
      ```

   

7. ### 查看远程库配置

   `不带参数时，就是列出已存在的远程分支`

   ```shell
   $ git remote
   origin
   ```

   

   `-v，--verbose` 查看所有远程仓库配置

   ```shell
   $ git remote -v
   origin  http://git.xxx.com/frameworks/acitvity.git (fetch)
   origin  http://git.xxx.com/frameworks/acitvity.git (push)
   ```

   

   `查看特定远程仓库的URL`

   ```shell
   $ git remote get-url origin
   http://git.xxx.com/frameworks/acitvity.git
   ```

   

8. ### 查看远程库信息以及和本地库的关系

   这个命令会联网去查询远程库信息，并且会列出和本地库的关系。

   `git remote show <name>`

   ```shell
   $ git remote show origin
   * remote origin
     Fetch URL: http://git.songker.com/frameworks/acitvity.git
     Push  URL: http://git.songker.com/frameworks/acitvity.git
     HEAD branch: master
     Remote branches:
       develop                     tracked
       master                      tracked
     Local branches configured for 'git pull':
       master     merges with remote master
     Local refs configured for 'git push':
       master           pushes to master           (up to date)
   
   ```

   

9. 

10. 