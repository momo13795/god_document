## Let's Encrypt 使用教程，免费的SSL证书，让你的网站拥抱 HTTPS

## Certbot 简介

[Certbot](https://certbot.eff.org/) 是Let’s Encrypt官方推荐的获取证书的客户端，可以帮我们获取免费的Let’s Encrypt 证书。Certbot 是支持所有 Unix 内核的操作系统的，个人博客的服务器系统是CentOS 7，这篇教程也是通过在个人博客上启用HTTPS的基础上完成的

###  获取免费证书

1. 安装Certbot客户端

   `yum install certbot`

2. 获取证书

   `certbot certonly --webroot -w /var/www/example -d example.com -d www.example.com`

>  *这个命令会为 example.com 和 www.example.com 这两个域名生成一个证书，使用* `--webroot` *模式会在* `/var/www/example` *中创建* `.well-known` *文件夹，这个文件夹里面包含了一些验证文件，certbot 会通过访问 example.com/.well-known/acme-challenge 来验证你的域名是否绑定的这个服务器。这个命令在大多数情况下都可以满足需求，*

但是有些时候我们的一些服务并没有根目录，例如一些微服务，这时候使用 `--webroot` 就走不通了。certbot 还有另外一种模式 `--standalone` ， 这种模式不需要指定网站根目录，他会自动启用服务器的443端口，来验证域名的归属。我们有其他服务（例如nginx）占用了443端口，就必须先停止这些服务，在证书生成完毕后，再启用。

`--webroot`简单来说需要先搭建nginx,apache这种服务器，才能生成证书

3. 直接获取证书

   `certbot certonly --standalone  -d lets.jucce.cn `

> 该命令验证域名所有权是通过80端口访问的，如80端口被暂用则证书生成失败

证书位置：/etc/letsencrypt/live/

```javascript
cert.pem  - Apache服务器端证书  
chain.pem  - Apache根证书和中继证书  
fullchain.pem  - Nginx所需要ssl_certificate文件  
privkey.pem - 安全证书KEY文件
Nginx环境，就只需要用到fullchain.pem和privkey.pem两个证书文件
```

## 更新证书

命令：`certbot renew`

证书是90天才过期，我们只需要在过期之前执行更新操作就可以了。 这件事情就可以直接交给定时任务来完成。linux 系统上有 `cron` 可以来搞定这件事情。 我新建了一个文件 `certbot-auto-renew-cron`， 这个是一个 `cron` 计划，这段内容的意思就是 每隔 两个月的 凌晨 2:15 执行 更新操作。

```
15 2 * */2 * certbot renew --pre-hook "service nginx stop" --post-hook "service nginx start"
```

`--pre-hook` 这个参数表示执行更新操作之前要做的事情，因为我有 `--standalone` 模式的证书，所以需要 停止 `nginx` 服务，解除端口占用。 `--post-hook` 这个参数表示执行更新操作完成后要做的事情，这里就恢复 `nginx` 服务的启用

最后我们用 `crontab` 来启动这个定时任务

```
crontab certbot-auto-renew-cron
```

## 解决证书更新提示 failed

certbot renew时出现类似的错误：
OCSP check failed for /xxx/cert1.pem (are we offline?)

经搜索原因：原因是 ocsp.int-x3.letsencrypt.org 的 cname 域名 a771.dscq.akamai.net 受到了干扰。
可以采用本地修改hosts的方案进行临时处理，在/etc/hosts中添加

```shell
23.32.3.72     ocsp.int-x3.letsencrypt.org
```

## 参考文献

[Let's Encrypt 使用教程](https://diamondfsd.com/lets-encrytp-hand-https/)

[Let's Encrypt 证书生成](https://ld246.com/article/1531709298417)