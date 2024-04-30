# GZIP

`gzip`是一种常用的网页压缩技术，传输的网页经过`gzip`压缩之后大小通常可以变为原来的一半甚至更小（官网原话），更小的网页体积也就意味着带宽的节约和传输速度的提升，特别是对于访问量巨大大型网站来说，每一个静态资源体积的减小，都会带来可观的流量与带宽的节省。

百度可以找到很多检测站点来查看目标网页有没有开启 `gzip` 压缩，随便找一个 [<网页GZIP压缩检测>](https://link.juejin.cn?target=http%3A%2F%2Ftool.chinaz.com%2FGzips%2FDefault.aspx%3Fq%3Djuejin.im) 输入掘金 `juejin.im` 来偷窥下掘金有没有开启 `gzip`。

这里可以看到掘金是开启了 `gzip` 的，压缩效果还挺不错，达到了 `52%` 之多，本来 `34kb` 的网页体积，压缩完只需要 `16kb`，可以想象网页传输速度提升了不少。



## 1.Nginx配置gzip

使用 `gzip` 不仅需要 `Nginx` 配置，浏览器端也需要配合，需要在请求消息头中包含 `Accept-Encoding: gzip`（IE5 之后所有的浏览器都支持了，是现代浏览器的默认设置）。

一般在请求 `html` 和 `css` 等静态资源的时候，支持的浏览器在 `request` 请求静态资源的时候，会加上 `Accept-Encoding: gzip` 这个 `header`，表示自己支持 `gzip` 的压缩方式。

`Nginx` 在拿到这个请求的时候，如果有相应配置，就会返回经过 `gzip` 压缩过的文件给浏览器，并在 `response` 相应的时候加上 `content-encoding: gzip` 来告诉浏览器自己采用的压缩方式（因为浏览器在传给服务器的时候一般还告诉服务器自己支持好几种压缩方式，例如：`gzip, deflate, br, zstd`），浏览器拿到压缩的文件后，根据自己的解压方式进行解析。

先来看看 `Nginx` 怎么进行 `gzip` 配置，和之前的配置一样，为了方便管理，还是在 `/etc/nginx/conf.d/` 文件夹中新建配置文件 `gzip.conf` ：

```bash
# /etc/nginx/conf.d/gzip.conf

gzip on; # 默认off，是否开启gzip
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

# 上面两个开启基本就能跑起了，下面的愿意折腾就了解一下
gzip_static on;
gzip_proxied any;
gzip_vary on;
gzip_comp_level 6;
gzip_buffers 16 8k;
# gzip_min_length 1k;
gzip_http_version 1.1;
```

稍微解释一下：

1. **gzip_types**：要采用 `gzip` 压缩的 `MIME` 文件类型，其中 `text/html` 被系统强制启用；

2. **gzip_static**：默认 `off`，该模块启用后，`Nginx` 首先检查是否存在请求静态文件的 `gz` 结尾的文件，如果有则直接返回该 `.gz` 文件内容；

3. **gzip_proxied**：默认 `off`，`nginx`作为反向代理时启用，用于设置启用或禁用从代理服务器上收到相应内容 `gzip` 压缩；

4. **gzip_vary**：用于在响应消息头中添加 `Vary：Accept-Encoding`，使代理服务器根据请求头中的 `Accept-Encoding` 识别是否启用 `gzip` 压缩；

5. **gzip_comp_level**：`gzip` 压缩比，压缩级别是 `1-9`，`1` 压缩级别最低，`9` 最高，级别越高压缩率越大，压缩时间越长，建议 `4-6`；

6. **gzip_buffers**：获取多少内存用于缓存压缩结果，`16 8k` 表示以 `8k*16` 为单位获得；

7. **gzip_min_length**：允许压缩的页面最小字节数，页面字节数从`header`头中的 `Content-Length` 中进行获取。默认值是 0，不管页面多大都压缩。建议设置成大于 `1k` 的字节数，小于 `1k` 可能会越压越大；

8. **gzip_http_version**：默认 `1.1`，启用 `gzip` 所需的 `HTTP` 最低版本；

   这个配置可以插入到 `http` 模块整个服务器的配置里，也可以插入到需要使用的虚拟主机的 `server` 或者下面的 `location` 模块中，当然像上面我们这样写的话就是被 `include` 到 `http` 模块中了。



其他更全的配置信息可以查看 [<官网文档ngx_http_gzip_module>](https://link.juejin.cn?target=http%3A%2F%2Fnginx.org%2Fen%2Fdocs%2Fhttp%2Fngx_http_gzip_module.html)，配置前是这样的：

```html
Request Headers
Accept-Encoding: gzip, deflate
```

配置之后 `response` 的 `header` 里面多了一个 `Content-Encoding: gzip`，返回信息被压缩了：

```html
Response Headers
Content-Encoding: gzip
```



**注意**，一般 `gzip` 的配置建议加上 `gzip_min_length 1k`，不加的话：

由于文件太小，`gzip` 压缩之后得到了 `-48%` 的体积优化，压缩之后体积还比压缩之前体积大了，所以最好设置低于 `1kb` 的文件就不要 `gzip` 压缩了。



## 2.Webpack的gzip配置

当前端项目使用 `Webpack` 进行打包的时候，也可以开启 `gzip` 压缩：

```javascript
// vue-cli3 的 vue.config.js 文件
const CompressionWebpackPlugin = require('compression-webpack-plugin')

module.exports = {
  // gzip 配置
  configureWebpack: config => {
    if (process.env.NODE_ENV === 'production') {
      // 生产环境
      return {
        plugins: [new CompressionWebpackPlugin({
          test: /\.js$|\.html$|\.css/,    // 匹配文件名
          threshold: 10240,               // 文件压缩阈值，对超过10k的进行压缩
          deleteOriginalAssets: false     // 是否删除源文件
        })]
      }
    }
  },
  ...
}
```

这里可以看到某些打包之后的文件下面有一个对应的 `.gz` 经过 `gzip` 压缩之后的文件，这是因为这个文件超过了 `10kb`，有的文件没有超过 `10kb` 就没有进行 `gzip` 打包，如果你期望压缩文件的体积阈值小一点，可以在 `compression-webpack-plugin` 这个插件的配置里进行对应配置。

那么为啥这里 `Nginx` 已经有了 `gzip` 压缩，`Webpack` 这里又整了个 `gzip` 呢，因为如果全都是使用 `Nginx` 来压缩文件，会耗费服务器的计算资源，如果服务器的 `gzip_comp_level` 配置的比较高，就更增加服务器的开销，相应增加客户端的请求时间，得不偿失。

如果压缩的动作在前端打包的时候就做了，把打包之后的高压缩等级文件作为静态资源放在服务器上，`Nginx` 会优先查找这些压缩之后的文件返回给客户端，相当于把压缩文件的动作从 `Nginx` 提前给 `Webpack` 打包的时候完成，节约了服务器资源，所以一般**推介在生产环境应用 `Webpack` 配置 `gzip` 压缩**。