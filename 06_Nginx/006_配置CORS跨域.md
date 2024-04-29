# CORS跨域

要解决跨域问题，我们来制造一个跨域问题。

首先设置 `x.lsq.club` 和 `y.lsq.club` 二级域名，都指向本云服务器地址。

**虽然对应 IP 是一样的**，但是在 `x.lsq.club` 域名发出的请求访问 `y.lsq.club` 域名的请求还是跨域了，**因为访问的 host 不一致。**



## 1.使用反向代理解决跨域

在前端服务地址为 `x.lsq.club` 的页面请求 `y.lsq.club` 的后端服务导致的跨域，可以这样配置：

```bash
server {
  listen 9001;
  server_name x.lsq.club;

  location / {
    proxy_pass y.lsq.club;
  }
}
```

这样就将对前一个域名 `x.lsq.club` 的请求全都代理到了 `y.lsq.club`，前端的请求都被我们用服务器代理到了后端地址下，绕过了跨域。



这里对**静态文件**的请求和**后端服务**的请求都以 `x.lsq.club` 开始，不易区分，所以为了实现对后端服务请求的统一转发，通常我们会**约定：**

对后端服务的请求加上 `/apis/` 前缀或者其他的 path 来和对静态资源的请求加以区分。

此时我们可以这样配置：

```bash
# 请求跨域，约定代理后端服务请求path以/apis/开头
location ^~/apis/ {
    # 这里重写了请求，将正则匹配中的第一个分组的path拼接到真正的请求后面，并用break停止后续匹配
    rewrite ^/apis/(.*)$ /$1 break;
    proxy_pass x.lsq.club;
  
    # 两个域名之间cookie的传递与回写
    proxy_cookie_domain x.lsq.club y.lsq.club;
}
```



这样，静态资源我们使用 `x.lsq.club/xx.html`，动态资源我们使用 `x.lsq.club/apis/getAwo`，浏览器页面看起来仍然访问的前端服务器，绕过了浏览器的同源策略，毕竟我们看起来并没有跨域。



也可以统一一点，直接把前后端服务器地址直接都转发到另一个 `server.lsq.club`，只通过在后面添加的`path`来区分请求的是静态资源还是后端服务，看需求了。



## 2.配置 header 解决跨域

当浏览器在访问跨源的服务器时，也可以在跨域的服务器上直接设置`Nginx`，从而前端就可以无感地开发，不用把实际上访问后端的地址改成前端服务的地址，这样可适性更高。

比如前端站点是 `x.lsq.club`，这个地址下的前端页面请求 `y.lsq.club` 下的资源，比如前者的 `x.lsq.club/index.html` 内容是这样的：

```html
<html>
<body>
    <h1>welcome x.lsq.club!!<h1>
    <script type='text/javascript'>
    var xmlhttp = new XMLHttpRequest()
    xmlhttp.open("GET", "http://y.lsq.club/index.html", true);
    xmlhttp.send();
    </script>
</body>
</html>
```

打开浏览器访问 `y.lsq.club/index.html` 的结果如下：

```html
welcome x.lsq.club!!
```



很明显这里是跨域请求，在浏览器中直接访问 `http://y.lsq.club/index.html` 是可以访问到的，但是在 `x.lsq.club` 的 html 页面访问就会出现跨域。



在 `/etc/nginx/conf.d/` 文件夹中新建一个配置文件，对应二级域名 `y.lsq.club` ：

```bash
# /etc/nginx/conf.d/y.lsq.club.conf

server {
  listen       80;
  server_name  y.lsq.club;
  
	add_header 'Access-Control-Allow-Origin' $http_origin;   # 全局变量获得当前请求origin，带cookie的请求不支持*
	add_header 'Access-Control-Allow-Credentials' 'true';    # 为 true 可带上 cookie
	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';  # 允许请求方法
	add_header 'Access-Control-Allow-Headers' $http_access_control_request_headers;  # 允许请求的 header，可以为 *
	add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
	
  if ($request_method = 'OPTIONS') {
		add_header 'Access-Control-Max-Age' 1728000;   # OPTIONS 请求的有效期，在有效期内不用发出另一条预检请求
		add_header 'Content-Type' 'text/plain; charset=utf-8';
		add_header 'Content-Length' 0;
    
		return 204;                  # 200 也可以
	}
  
	location / {
		root  /usr/share/nginx/html/be;
		index index.html;
	}
}
```



然后 `nginx -s reload` 重新加载配置。这时再访问 `x.lsq.club/index.html` 结果如下，请求中出现了我们刚刚配置的`Header`，解决了跨域问题。

