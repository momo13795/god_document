---
typora-copy-images-to: upload
---

# ssh 常见知识

## 1.公钥私钥生成

`命令：`

```shell
ssh-keygen -t rsa -C "xxx@qq.com" -f ~/.ssh/id_rsa_yangan
```

> **-C:**表示的是后面的标识一般填一个邮箱即可
>
> **-f: ** 表示的是公钥私钥存放的位置

生成结果如下：

![Snipaste_2020-09-11_16-06-11](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/Snipaste_2020-09-11_16-06-11.png)

## 2. ssh-config 使用

`附上常用配置:`

``` shell
###############bitbucket 配置##############################
 Host person
   HostName bitbucket.org
   User git
   Preferredauthentications publickey
   IdentityFile ~/.ssh/id_rsa_bitbucket

###############bitbucket 配置##############################

##############linux服务器配置##########################
Host web1
    Hostname ***
    User root
    Port 22
    IdentityFile ~/.ssh/id_rsa_linux

#vaffle_test
 Host vaffle_test
   Hostname ****
   User root
   Port 22
   IdentityFile ~/.ssh/id_rsa_linux
##############linux服务器配置##########################
```

`对ssh-config 参数说明：`

> **Host：**连接的别名
>
> **HostName：** 主机域名或ip
>
> **port：**连接端口号(一般是22)
>
> **IdentityFile：**私钥路径
>
> **User：**连接用户
>
> **ServerAliveInterval：**每隔幾秒發送 keep-alive 保持連線