# Let‘s Encrypt

### 1.Let‘s Encrypt是什么？

`Let's Encrypt`是一个由`Internet Security Research Group (ISRG)`运营的非营利性证书颁发机构`（CA）`，致力于为全球网站提供免费的`SSL/TLS`证书，从而推广`HTTPS`加密，提升网络安全性。

#### Let's Encrypt的基本信息

- **免费**：Let’s Encrypt的最大特点是提供免费的SSL/TLS证书，这使得任何网站都可以免费获得HTTPS加密，并增加用户数据的安全性。
- **短期证书**：Let’s Encrypt颁发的证书有效期为90天，相较于传统CA颁发的1年或更长期的证书，需要更频繁地更新证书。但自动化续期和更新过程相对简便。

- **自动化**：Let’s Encrypt采用了自动化的证书颁发流程，通过简单的命令或API可以轻松获取证书。证书的申请、验证和安装过程大部分都是自动化完成的。



### 2.Certbot工具

`Certbot` 是一个由 `Electronic Frontier Foundation（EFF）`支持的***免费、开源的工具，用于自动化获取、部署和更新 HTTPS 证书***。它是为了方便使用 `Let’s Encrypt` 服务而开发的，允许用户在 Web 服务器上快速设置 SSL/TLS 加密连接。

**Certbot 的主要特点和功能**

- **自动化证书获取和更新:** Certbot 提供了简单的命令行接口，用于自动获取、安装和更新 Let’s Encrypt 的 SSL/TLS 证书。
- **证书管理和续期**: Certbot 还提供了自动的证书续期功能，它会在证书接近到期时自动更新证书，确保您的网站持续地使用最新的有效证书。
- **支持多种服务器和操作系统**: Certbot 支持多种主流的 Web 服务器，包括 Apache、Nginx、Microsoft IIS 等，并且适用于各种常见的 Linux 发行版。
- **交互式操作**: Certbot 提供了用户友好的交互式界面，使证书的获取和安装过程更加直观和简单。



**使用`Docker`容器安装`Certbot`**

1. ##### 拉取`Certbot`镜像

   ```shell
   docker pull certbot/certbot:latest
   ```

   拉取下来的镜像如下：

   ```shell
   [root@iZwz99dsk8ug1u165edwzsZ docker]# docker images
   REPOSITORY        	TAG                 IMAGE ID            CREATED             SIZE
   certbot/certbot   	latest              6ddd9ccfe97c        3 years ago         111MB
   ```

   

2. ##### 配置 `docker-compose.yml`，创建`Certbot`容器，配置内容如下：

   ```shell
     certbot:
       container_name: certbot
       image: certbot/certbot:latest
       volumes:
         - ./nginx/cert/certbot/www:/usr/share/certbot/www:rw #http验证目录，可设置rw可写，与nginx容器对应的宿主机目录是一致的
         - ./nginx/cert/certbot/ssl:/etc/letsencrypt #证书位置，同上，注意不要只映射到live，而是它的上一级
   ```

   **注意：** 此处需要配置<font color="red">验证目录</font>和生成的<font color="red">证书存储目录</font>。

   

   创建容器`docker-compose up -d certbot`：

   ```shell
   CONTAINER ID	IMAGE					COMMAND		CREATED			STATUS					PORTS			NAMES
   2aa03868dd9c	certbot/certbot:latest  "certbot"   7 weeks ago     Exited (1) 6 weeks ago  				certbot
   ```



### 3.证书的申请操作步骤

1. ##### 把域名DNS解析到服务器IP上

2. ##### 在`nginx`上添加验证配置（**生成证书时需要验证服务器上的该目录**）

   ```shell
   server {
   	listen 80;
   	server_name gw.songker.com; # 此处改为自己的域名(以gw.songker.com为例)
   	
   	# 配置http验证可访问
       location /.well-known/acme-challenge/ {
           #此目录都是nginx容器内的目录，对应宿主机volumes中的http验证目录，而宿主机的又与certbot容器中命令--webroot-path指定目录一致，从而就整个串起来了，解决了http验证问题
           root /etc/nginx/cert/certbot/www/;
       }
   }
   ```

   `certbot`必须要和`nginx`搭配，因为在申请证书前会进行域名验证，就需要在你的站点下放一个`/.well-known/acme-challenge`，这种方式就是官网的`Webroot`的方式。

   

   修改配置后重启nginx。

   

3. ##### 申请证书

   执行如下命令：

   ```shell
   docker-compose -f docker-compose.yml run --rm  certbot certonly --webroot --webroot-path /usr/share/certbot/www/ -d gw.songker.com
   ```

   第一次运行会提示你填写邮箱地址，随便填。

   

   执行完之后，将会在`./nginx/cert/certbot/ssl`目录下看到五个文件夹：`accounts、archive、live、renewal、renewal-hooks`。

   ```shell
   [root@iZwz99dsk8ug1u165edwzsZ docker]# ll nginx/cert/certbot/ssl/
   total 28
   drwx------  3 root root 4096 Oct 16 11:09 accounts
   drwx------ 12 root root 4096 Jan  8 11:09 archive
   drwx------ 12 root root 4096 Jan  8 11:09 live
   drwxr-xr-x  2 root root 4096 Jan  8 11:09 renewal
   drwxr-xr-x  5 root root 4096 Oct 16 10:56 renewal-hooks
   ```

   我们`nginx`需要用到的是`live`目录下的文件。

   ```shell
   [root@iZwz99dsk8ug1u165edwzsZ docker]# ll nginx/cert/certbot/ssl/live/gw.songker.com/
   total 4
   lrwxrwxrwx 1 root root  38 Dec 16 03:04 cert.pem -> ../../archive/gw.songker.com/cert2.pem
   lrwxrwxrwx 1 root root  39 Dec 16 03:04 chain.pem -> ../../archive/gw.songker.com/chain2.pem
   lrwxrwxrwx 1 root root  43 Dec 16 03:04 fullchain.pem -> ../../archive/gw.songker.com/fullchain2.pem
   lrwxrwxrwx 1 root root  41 Dec 16 03:04 privkey.pem -> ../../archive/gw.songker.com/privkey2.pem
   -rw-r--r-- 1 root root 692 Oct 16 11:27 README
   ```

   主要关注以下两个：

   - `live/gw.songker.com/fullchain.pem`对应着`nginx`的`ssl_certificate`配置
   - `live/gw.songker.com/privkey.pem`对应着`nginx`的`ssl_certificate_key`配置

   

   以上是申请证书的全部过程。



### 4.证书的使用配置

在`nginx`配置文件中添加`https`配置：

```shell
server {
    listen 443 ssl http2;
    ssl_certificate cert/certbot/ssl/live/gw.songker.com/fullchain.pem;		# 此处是生成证书保存的目录
    ssl_certificate_key cert/certbot/ssl/live/gw.songker.com/privkey.pem;	# 此处是生成证书保存的目录
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    server_name    gw.songker.com;

    client_header_buffer_size    256k;
    large_client_header_buffers  4 1m;

    location / {
       proxy_pass http://localhost:9080;
       proxy_set_header   Host    $host;
       proxy_set_header   X-Real-IP   $remote_addr;
       proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
       client_max_body_size 200m;
    }
}
```



### 5.证书的自动续签配置

​	`Certbot` 的证书有效期为 90 天，需要设置自动续订。

1. ##### 新建一个`certbot-crontab.sh`

   ```bash
   #！/bin/bash
   
   echo "certbot检查：start"
   docker-compose -f /mnt/docker/docker-compose.yml run --rm certbot renew
   echo "certbot检查：end"
   
   sleep 10
    
   echo "重新加载nginx：start"
   docker restart nginx
   echo "重新加载nginx：end"
   ```

2. ##### 增加`certbot-crontab.sh`的权限为可执行

   ```shell
   chomd +x certbot-crontab.sh
   ```

3. ##### 使用`linux`的`crontab`定时执行，指令如下：

   ```shell
   crontab -e
   ```

   将以下内容复制上去，其中`certbot-crontab.sh`、`crontab-log.txt`的路径替换成你自己的。

   ```shell
   # 每隔3天凌晨3点执行一次该任务
   0 3 */3 * * /mnt/docker/certbot/certbot-crontab.sh > /mnt/docker/certbot/log/crontab-log.txt 2>&1 &
   ```

   



