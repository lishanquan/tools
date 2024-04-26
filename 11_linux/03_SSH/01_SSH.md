1. ### 免密登录配置步骤

   1. 准备两台安装了CentOS7的主机，一台作为客户端，一台作为服务端。客户端主机需要安装openssh-clients。

      ```shell
      yum install -y openssh-clients
      ```

   2. **客户端执行 ssh-keygen 命令生成公钥与私钥。**

      默认使用RSA非对称加密算法，此命令会询问秘钥对存放位置，按回车使用默认即可，

      提示输入passphrase也是按回车使用默认即可（按两次回车）。

      

      生成的密钥对默认存放在 当前用户的家目录/.ssh目录（**<u>/root/.ssh</u>**）中，

      id_ras是私钥，id_rsa.pub是公钥。

      需要将id_rsa.pub公钥内容追加到服务端的 authorized_keys 文件中。

   3. **将生成的id_rsa.pub公钥内容追加到服务端**

      客户端执行 ssh-copy-id root@服务端IP 将本机的id_rsa.pub公钥内容追加到服务端的/root/.ssh/authorized_keys 文件中。

      然后提示需要输入服务端root用户的密码，输入密码，回车即可。

      

      **注**：也可以拷贝客户端的id_rsa.pub公钥内容，然后追加到服务端的/root/.ssh/authorized_keys文件中，与执行 ssh-copy-id root@服务端IP 命令是同样的效果。

   4. 客户端执行 ssh root@服务端IP ，就直接登录到服务端了。

2. ### 远程登录

   ```shell
   ssh [用户名]@[IP地址]
   ```

3. ### 执行远程命令

   ```shell
   ssh [用户名]@[IP地址] [命令]
   ```

4. ### 文件传输

   ```shell
   scp [本地文件路径] [用户名]@[IP地址]:[目标路径]
   ```

   