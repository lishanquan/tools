# 配置HTTPS

具体配置过程网上挺多的了，也可以使用你购买的某某云，一般都会有[免费申请](https://link.juejin.cn?target=https%3A%2F%2Fcloud.tencent.com%2Fdocument%2Fproduct%2F400%2F6814)的服务器证书，安装直接看所在云的操作指南即可。

我购买的腾讯云提供的亚洲诚信机构颁发的免费证书只能一个域名使用，二级域名什么的需要另外申请，但是申请审批比较快，一般几分钟就能成功，然后下载证书的压缩文件，里面有个 `nginx` 文件夹，把 `xxx.crt` 和 `xxx.key` 文件拷贝到服务器目录，再配置下：



```nginx
server {
  listen 443 ssl http2 default_server;   # SSL 访问端口号为 443
  server_name lsq.club;         # 填写绑定证书的域名

  ssl_certificate /etc/nginx/https/1_lsq.club_bundle.crt;   # 证书文件地址
  ssl_certificate_key /etc/nginx/https/2_lsq.club.key;      # 私钥文件地址
  ssl_session_timeout 10m;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;      #请按照以下协议配置
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE; 
  ssl_prefer_server_ciphers on;
  
  location / {
    root         /usr/share/nginx/html;
    index        index.html index.htm;
  }
}
```



写完 `nginx -t -q` 校验一下，没问题就 `nginx -s reload`，现在去访问 `https://lsq.club/` 就能访问 `HTTPS` 版的网站了。

一般还可以加上几个增强安全性的命令：



```nginx
add_header X-Frame-Options DENY;           # 减少点击劫持
add_header X-Content-Type-Options nosniff; # 禁止服务器自动解析资源类型
add_header X-Xss-Protection 1;             # 防XSS攻击
```
