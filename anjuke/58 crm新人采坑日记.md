## 58 crm新人采坑日记

> 采坑人：徐剑
>
> 采坑时间：2021-05-15
>
> 本地环境：macOS Big Sur 11.0  M1

### 1.crm 本地环境搭建

1. `php 版本：测试环境版本为7.0  用其他的版本注意一些代码的支持，`，<!--composer 2.0 安装的扩展会去比对扩展的php版本和运行的php版本（直接用test 测试环境直接做开发 需要关闭composer.json config 下的"platform-check": false）-->、

2. `php 扩展：`

   > `brew install apcu yac` <!--composer update 需要这些扩展-->
   
3. `php 扩展之memcache：` <!--test 环境版本3.0.9-->

   ```
   ###先安装memcache 需要的依赖
   brew install pkg-config
   brew install zlib 
   brew install libmemcached
   ```

   php扩展url:`http://pecl.php.net/package/memcache`

   ```
   curl -O http://pecl.php.net/get/memcache-4.0.5.1.tgz
   mkdir memcache-4.0.5.1
   tar -zvxf memcache-4.0.5.1.tgz  -C  ~/Download/memcache-4.0.5.1
   cd memcache-4.0.5.1/memcache-4.0.5.1
   phpize
   ./configure --with-zlib-dir=/opt/homebrew/opt/zlib  (该地方替换自己zlib的目录)
   make
   sudo make install
   ```

   ------

   `注意点：安装完扩展需要在php.ini 引入此扩展 并重启php-fpm`

   

4. `php 扩展之memcached：` <!--test 环境版本3.0.4-->

   php扩展url:`http://pecl.php.net/package/memcached`

   ```
   同第3步骤安装
   ```

   

5. `php 扩展reids` <!--test 环境版本3.0.0-->

   php扩展url:`http://pecl.php.net/package/redis`

   ```
   同第3步骤安装
   ```
6. `pecl 安装的yac没有开启 yac.enable_cli 需要 在php.ini里面开启 yac.enable_cli = 1` 

## 2.vpn 环境下注意点

### 2.1 绑定host

```
##开vpn需要绑定host

##scm 接口
10.249.7.81 xujian13.scrm.new.kfs.dev.anjuke.test

#db 地址
10.249.7.82 db.kfs.dev.anjuke.test

#redis cookie配置
10.249.7.81 kfs.dev.anjuke.test
##开vpn需要绑定host
```

