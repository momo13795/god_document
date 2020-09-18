## Mac 电脑不自动休眠排查问题

1. 打开活动监视器

![Snipaste_2020-09-18_10-25-25](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/Snipaste_2020-09-18_10-25-25.png)

查看是否有防止睡眠的程序。

2. 开启Mac 终端模式

命令：`pmset -g`

![Snipaste_2020-09-18_10-28-46](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/Snipaste_2020-09-18_10-28-46.png)

​					图1

> `sleep:`电脑进入休眠的时间 ，单位是：分钟
>
> `displaysleep:` 显示器关闭的时间，单位是：分钟
>
> `disksleep:`硬盘进入休眠的时间，单位是：秒
>
> 注意点：sleep>=displaysleep>=disksleep

3. 解决方法

如上图1来看，我这边有两个程序阻止电脑进入休眠。

1. Chrmoe 浏览器。排查 了一下，是我后台暂停了一个未下载完的文件导致，关闭后即可解决谷歌阻止休眠

   ![Snipaste_2020-09-18_10-35-51](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/Snipaste_2020-09-18_10-35-51.png)
   
2. 排查图上还有一个分享阻止休眠的方法
   
   系统设置->共享  
   
   关闭共享即可
   
   ![Snipaste_2020-09-18_10-40-23](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/Snipaste_2020-09-18_10-40-23.png)