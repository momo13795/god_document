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

## 3. Ssh 远程执行命令

日常情况我们需要到服务器上去执行一些命令或拷贝文件，本来是很简单的一件事情，但做得多了就比较繁琐了。通过 SSH 帮我们远程执行相关命令或脚本，非常棒。

1. 执行远程命令：

> ```shell
> $ ssh example "cd /; ls"
> bin
> boot
> data
> ```

2. 执行多行命令：

   > ~~~ shell
   > ssh example "
   > dquote> cd / 
   > dquote> ls
   > dquote> "
   > bin
   > boot
   > data
   > ...
   > ~~~

3. 执行本地脚本 有时候，我们会将经常用到命令，如将一个复杂命令写入到一个 shell 脚本中，但又不想拷贝到服务器上，那么：

   > ```shell
   > # 示例脚本
   > $ echo "cd /; ls" > test.sh
   > # 为脚本添加可执行权限
   > $ chmod +x test.sh
   > 
   > # 远程执行本地脚本
   > $ ssh example < test.sh
   > Pseudo-terminal will not be allocated because stdin is not a terminal.
   > bin
   > boot
   > data
   > ...
   > ```

4. 执行需要交互的命令：

   > ```shell
   > ssh -t example "top"
   > ```