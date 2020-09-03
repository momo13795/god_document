# docker mailu 邮件搭建

## 1.先决条件

> 1. 要有一台vps
> 2. 要有一个域名
> 3. Docker docker-compose 环境

## 2 准备配置文件

[配置地址](https://mailu-setup.kanojo.de/)

### 2.1 step1

![1](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/1.png)

### 2.2 step 2

![2](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/2.png)

![3](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/3.png)

### 2.3 step 3

![4](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/4.png)

### 2.4 step 4

![](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/5.png)

### 2.5 step 5

![](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/6.png)

### 2.6 配置生成结果

![](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/7.png)

## 3 按配置步骤操作

![Snipaste_2020-09-02_14-40-49](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/Snipaste_2020-09-02_14-40-49.png)

## 4 配置dns解析

### 4.1 登录邮箱后台

地址：`http://abc.com/admin`

![10](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/10.png)

### 4.2 创建dns

![11](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/11.png)

![12](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/12.png)

![13](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/13.png)

![14](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/14.png)

## 5 注意问题

1. 解决vps 已经绑定25端口的问题

   ![9](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/9.png)

解决办法：

`netstat -lnp |grep :25`

`kill -9 1066`

## 6 解决gcp 出口端口25的问题

**解决思路：**postfix 配合mailgun 做中继服务器

### 第一种shell

``` shell
#!/bin/bash


# Configuration for the script
POSTFIX_CONFIG=/etc/postfix/main.cf
POSTFIX_SASL=/etc/postfix/sasl_passwd

function confirm () {
  read -r -p "${1:-Are you sure? [Y/n]} " response
  if [[ $response == "" || $response == "y"  || $response == "Y" ]]; then
    echo 0;
  else
    echo 1;
  fi
}

# Check that we're root, otherwise exit!
if [ "$(id -u)" != "0" ]; then
   echo "This script must be run as root!" 1>&2
   exit 1
fi

# Read some configuration input
read -r -p "Enter your Mailgun SMTP login: " SMTP_LOGIN
read -r -p "Enter your Mailgun SMTP password: " SMTP_PASSWORD

CONTINUE=`confirm "Continue with $SMTP_LOGIN:$SMTP_PASSWORD? [Y/n] "`
if [[ CONTINUE -ne 0 ]]; then
  echo "Aborting...";
  exit;
fi

# Set a safe umask
umask 077

# Install postfix and friends
apt-get update && apt-get install postfix libsasl2-modules -y

# Comment out a couple transport configurations
sed -i.bak "s/default_transport = error/# default_transport = error/g" $POSTFIX_CONFIG
sed -i.bak "s/relay_transport = error/# relay_transport = error/g" $POSTFIX_CONFIG
sed -i.bak "s/relayhost =/# relayhost =/g" $POSTFIX_CONFIG

# Add the relay host for Mailgun, force SSL/TLS, and other config
#以强制支持 SSL/TLS 并为这些类型的请求配置 SMTP 身份验证功能。由一个简单的访问和安全层 (SASL) 模块负责处理 Postfix 配置中的身份验证。将以下几行代码添加到文件末尾
cat >> $POSTFIX_CONFIG << EOF

# Mailgun configuration
relayhost = [smtp.mailgun.org]:2525
smtp_tls_security_level = encrypt
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
EOF

# Authentication stuff
cat > /etc/postfix/sasl_passwd << EOF
[smtp.mailgun.org]:2525 YOUR_SMTP_LOGIN:YOUR_SMTP_PASSWORD
EOF

# 使用 postmap 实用程序生成一个 .db 文件 
postmap $POSTFIX_SASL
#移除包含您凭据的文件，因为将不再需要该文件。
rm $POSTFIX_SASL

# 设置您 .db 文件的权限
chmod 600 $POSTFIX_SASL.db

# Reload Postfix
postfix reload
```



**使用方法：**

``` shell
sudo chmod +x relay_conf.sh 
sudo docker cp ./relay_conf.sh mailu_smtp_1:/relay_conf.sh 
sudo docker exec  -it  mailu_smtp_1 /relay_conf.sh
```

### 6.2 第二种shell

1. 添加中继邮箱账号(这里使用的是mailgun)

``` shell
cd /mailu
cat >> mailu.env << EOF
RELAYHOST=[smtp.mailgun.org]:587
RELAY_LOGIN=xxx
RELAY_PASSWORD=xxx
EOF
```

2. 重启服务

`docker-compose up -d`

3. relay_conf.sh 内容

``` shell
#!/bin/sh

# Configuration for the script
POSTFIX_CONFIG=/etc/postfix/main.cf
POSTFIX_SASL=/etc/postfix/sasl_passwd

# Set a safe umask
umask 077

# Add the relay host params 
cat >> $POSTFIX_CONFIG << EOF
# Mailgun configuration
relayhost = $RELAYHOST
smtp_tls_security_level = encrypt
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:$POSTFIX_SASL
smtp_sasl_security_options = noanonymous
EOF

# Remove old
rm -f $POSTFIX_SASL

# Create new
cat > $POSTFIX_SASL << EOF
$RELAYHOST $RELAY_LOGIN:$RELAY_PASSWORD
EOF

chmod 600 $POSTFIX_SASL
postmap $POSTFIX_SASL

# Reload Postfix
postfix reload
```

4. **使用方法**

``` shell
sudo chmod +x relay_conf.sh 
sudo docker cp ./relay_conf.sh mailu_smtp_1:/relay_conf.sh 
sudo docker exec  -it  mailu_smtp_1 /relay_conf.sh
```

## 7 发送邮件

**访问地址：**https://你的域名/admin

 **登录账户：** admin@你的域名

![15](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/15.png)

