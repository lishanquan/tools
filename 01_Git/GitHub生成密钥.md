# Github生成密钥

Git可以通过https和ssh两种方式连接服务器。

https：配置简单，但是每次pull和push都需要输入账号和密码。

ssh：传输前压缩数据，传输效率高，不需要每次提供账号密码。

## 1. Git的username和email设置

```shell
$ git config --global user.name "yourusername"
$ git config --global user.email "youremailname@qq.com"
```

## 2. 生成密钥

使用注册Github的邮箱生成秘钥

```shell
ssh-keygen -t rsa -C "emailname@qq.com"
```

.ssh目录会生成id_rsa和id_rsa.pub两个文件，id_rsa是私钥，id_rsa.pub是公钥。

## 3. 添加SSH Key到Github账户

使用刚才生成的公钥，在Github账户中添加SSH Key。

## 4. 测试SSH Key是否设置成功

```shell
$ ssh -T git@github.com

The authenticity of host 'github.com (192.30.253.113)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
```

输出如下，则表示通过：

```shell
Hi yourusername! You've successfully authenticated, but GitHub does not provide shell access.
```

## 5. 切换项目连接方式

修改关联的远程仓库的方法，有三种：

1）使用 git remote set-url 命令，更新远程仓库的 url

```shell
$ git remote set-url origin git@github.com:yourusername/project-name
```

此行命令（set-rul）修改的是项目中 .git （隐藏）文件夹下的config文件。

使用上面的命令将同时更新提取网址和推送网址。

2）先删除之前关联的远程仓库，再来添加新的远程仓库关联

```shell
# 删除关联的远程仓库
$ git remote remove <name>

# 添加新的远程仓库关联
$ git remote add <name> <url>
```

远程仓库的名称推荐使用默认的名称 origin。

3）直接修改项目目录下的 .git 目录中的 config 配置文件。