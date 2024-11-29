参考文档：https://blog.csdn.net/qq_30261927/article/details/141057621



1. #### 先做好域名dns解析到服务器ip上

2. #### nginx必须做好的配置

   ```shell
   server {
   	listen 80;
   	server_name xxxxx;#此处改为自己的域名
   	
   	#配置http验证可访问
       location /.well-known/acme-challenge/ {
           #此目录都是nginx容器内的目录，对应宿主机volumes中的http验证目录，而宿主机的又与certbot容器中命令--webroot-path指定目录一致，从而就整个串起来了，解决了http验证问题
           root /etc/nginx/cert/certbot/www/;
       }
   }
   ```

3. #### 启动nginx以及certbot，docker-compose.yml代码如下：

   ```shell
   version: '3'
   services:
     nginx:
       image: nginx:latest
       container_name: nginx
       restart: always
       ports:
         - "80:80"
         - "443:443"
       volumes:
         - ./nginx/nginx.conf:/etc/nginx/nginx.conf
         - ./nginx/cert:/etc/nginx/cert
         - ./nginx/conf.d:/etc/nginx/conf.d
         - ./nginx/logs:/var/log/nginx
         - ./nginx/html:/etc/nginx/html
       networks:
         - basic_net
     certbot:
       container_name: certbot
       image: certbot/certbot
       volumes:
             - ./nginx/cert/certbot/www:/usr/share/certbot/www #http验证目录，可设置rw可写，与nginx容器对应的宿主机目录时一致的
             - ./nginx/cert/certbot/ssl:/etc/letsencrypt #证书位置，同上，注意不要只映射到live，而是它的上一级
   networks:
     basic_net:
       driver: bridge
   ```

   **注意：**

   `certbot`必须要和`nginx`搭配，因为在申请证书前会进行域名验证，就需要在你的站点下放一个`/.well-known/acme-challenge`，这种方式就是官网的Webroot的方式。

   

4. #### 证书申请

   命令如下：

   ```shell
   docker-compose -f docker-compose.yml run --rm certbot certonly --webroot --webroot-path /usr/share/certbot/www/ -d xxx.com
   ```

   其中：xxx.com替换成你的域名。

   第一次运行会提示你填写邮箱地址，随便填。

   执行完之后，将会在`./nginx/cert/certbot/ssl`目录下看到五个文件夹：`accounts`、`archive`、`live`、`renewal`、`renewal-hooks`，我们`nginx`需要用到的是`live`目录下的文件。

   主要关注以下两个：

   1. `live/xxx.com/fullchain.pem`对应着`nginx`的`ssl_certificate`配置 
   2. `live/xxx.com/privkey.pem`对应着`nginx`的`ssl_certificate_key`配置

   

   ##### `certbot certonly`常用参数

   ###### 基本认证参数

   - `-d`, --domains <DOMAIN>：指定要为哪个域名获取证书。可以<font color="red">多次</font>使用这个选项来为多个域名获取证书。
   - `--email` <EMAIL>：提供联系电子邮件地址，以便在证书即将到期或者出现问题时接收通知。
   - `--agree-tos`：同意`Let's Encrypt`的服务条款。

   ###### 验证方法

   - `--webroot`：使用基于Web根目录的挑战方法来验证域名所有权。需要配合 -w, --webroot-path <PATH> 使用，指定Web根目录路径。
   - `--standalone`：让Certbot短暂地运行一个独立的Web服务器来完成验证过程。这通常用于端口80没有被其他服务占用的情况。
   - `--manual`：如果你需要进行手动DNS验证或其他形式的手动干预。
   - `--preferred-challenges` <CHALLENGE_TYPES>：指定首选的挑战类型，比如 http-01 或 dns-01。

   ###### 其他参数

   - `--dry-run`：执行一次模拟运行，以测试命令是否能够成功运行，但不会实际获取证书。
   - `--force-renewal`：强制重新颁发证书，即使现有证书还未过期。
   - `--renew-by-default`：默认情况下自动续订证书。
   - `--no-eff-email`：不注册电子前线基金会（EFF）的邮件列表。
   - `--configurator <PLUGIN>`：指定使用的插件，例如 nginx 或 apache，但与 certonly 一起使用时，这些插件只用于认证，不会修改配置。
   - `--pre-hook <COMMAND>` 和 `--post-hook <COMMAND>`：在认证前和认证后执行自定义脚本或命令。

   

   以上是申请证书的全部过程。

   

5. #### 设置自动续订

   `Certbot` 的证书有效期为 90 天，需要设置自动续订。

   可以通过创建一个 `cron job` 来自动运行续订命令：

   1. 创建定时任务脚本`certbot-crontab.sh`

      ```shell
      #！/bin/bash
       
      echo "certbot检查：start"
      docker-compose -f /mnt/docker/docker-compose.yml run --rm certbot renew
      echo "certbot检查：end"
      
      sleep 10
       
      echo "重新加载nginx：start"
      docker restart nginx
      echo "重新加载nginx：end"
      ```

   2. 增加`certbot-crontab.sh`的权限为可执行

      ```shell
      chomd +x certbot-crontab.sh
      ```

   3. 使用`linux`的`crontab`，指令如下

      ```shell
      crontab -e
      ```

      将以下内容复制上去，其中`certbot-crontab.sh`、`crontab-log.txt`的路径替换成你自己的

      ```shell
      0 0 1,15 * * /home/docker/workspace/basic/certbot/certbot-crontab.sh > /home/docker/workspace/basic/certbot/log/crontab-log.txt 2>&1 &
      ```

      