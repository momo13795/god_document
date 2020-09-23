## crontab 知识

## 1. 定义

crontab是一个命令，常见于Unix和类Unix的操作系统之中，用于设置周期性被执行的指令。该命令从标准输入设备读取指令，并将其存放于“crontab”文件中，以供之后读取和执行。与windows下的计划任务类似，当安装完成操作系统后，默认会安装此服务工具，并且会自动启动crond进程，crond进程每分钟会定期检查是否有要执行的任务，如果有要执行的任务，则自动执行该任务。其中：

1. cron是服务名称
2. crond是后台进程
3. crontab是计划任务表

`常见参数列表：`

```she
usage:  crontab [-u user] file
        crontab [-u user] [ -e | -l | -r ]
                (default operation is replace, per 1003.2)
        -e      (edit user's crontab)
        -l      (list user's crontab)
        -r      (delete user's crontab)
        -i      (prompt before deleting user's crontab)
        -s      (selinux context)
```

位置：

   日志文件: ll /var/log/cron*
    编辑文件： vim /etc/crontab    
    进程：ps -ef | grep crond 



## 格式

![Snipaste_2020-09-18_14-16-23](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/Snipaste_2020-09-18_14-16-23.png)

> 前四行是用来配置crond任务运行的环境变量
>
> 第一行SHELL变量指定了系统要使用哪个shell，这里是bash
>
> 第二行PATH变量指定了系统执行命令的路径
>
> 第三行MAILTO变量指定了crond的任务执行信息将通过电子邮件发送给root用户
>
> 如果MAILTO变量的值为空，则表示不发送任务执行信息给用户
>
> 第四行的HOME变量指定了在执行命令或者脚本时使用的主目录
>
> +++
>
> 星号（*）：代表所有可能的值，如month字段为星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
>
> 逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
>
> 中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
>
> 正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。



## crontab 分类

### 1. 系统crontab

位置：/etc/crontab

> MIN HOUR DOM MON DOW USER CMD

一行設定包含六個部分，各部分的意義如下：



| 欄位 | 說明             | 可設定的值                                                   |
| :--- | :--------------- | :----------------------------------------------------------- |
| MIN  | 分钟             | `0` 到 `59`                                                  |
| HOUR | 小时             | `0` 到 `23`                                                  |
| DOM  | 日               | `1` 到 `31`                                                  |
| MON  | 月份             | `1` 到 `12`，可以用英文代替                                  |
| DOW  | 星期几           | `0`（周日）到 `6`（周日六），`7` 也代表周日。此欄位亦可用英文簡稱取代，例如週日也可以寫 `Sun`。 |
| USER | 执行的用户       | 如：root                                                     |
| CMD  | 要定期执行的命令 | 任何可執行的程式或指令稿（包含參數），例如 `/path/to/cmd --your --parameter`。 |

### 2. 个人crontab

在個人的 `crontab` 設定中，基本上每一行設定就代表一個要定期執行的程式，其格式如下：

> MIN HOUR DOM MON DOW CMD

## crontab 特殊排程规则

| 排程規則    | 說明                               |
| :---------- | :--------------------------------- |
| `@reboot`   | 每次重新開機之後，執行一次。       |
| `@yearly`   | 每年執行一次，亦即 `0 0 1 1 *`。   |
| `@annually` | 每年執行一次，亦即 `0 0 1 1 *`。   |
| `@monthly`  | 每月執行一次，亦即 `0 0 1 * *`。   |
| `@weekly`   | 每週執行一次，亦即 `0 0 * * 0`。   |
| `@daily`    | 每天執行一次，亦即 `0 0 * * *`。   |
| `@hourly`   | 每小時執行一次，亦即 `0 * * * *`。 |

例如每天執行一次，就可以這樣寫：

```shell
# 每天執行一次
@daily /home/gtwang/script.sh --your --parameter
```

## cron 一些知识

###### cron计划任务默认root用户与非root用户都可以执行，当然如果在安全方面想禁用这部分用户，则可以通过两个文件来解决：

1. cron.allow：定义允许使用crontab命令的用户 

2. cron.deny：定义拒绝使用crontab命令的用户

   ```she
   案例1：只允许root和www用户执行crontab命令，其他用户则禁止执行 
   在需要定义该策略的主机上面执行： 
   echo “www” > /etc/cron.allow
   ```

3. `ls /etc | grep cron*`

   ![Snipaste_2020-09-18_14-46-43](https://cdn.jsdelivr.net/gh/momo13795/upic_picture@master/uPic/Snipaste_2020-09-18_14-46-43.png)

`cron.daily`,``cron.hourly``,``cron.weekly,``,`cron.monthly`

这几个文件夹放的都是定时执行的脚本，列如每天，每周，每月

`原理：`

在CeontOS6 里面，crond每分钟去/etc/cron.d里面搜索配置文件，里面有一个0hourly文件，里面写了01 * * * * root run-parts /etc/cron.hourly。是每隔1小时去运行一次/etc/cron.hourly目录，该目录下面有一个0anacron文件，这样0anacron文件就能每小时运行一次。0anacron按照/etc/anacrontab文件里面的配置，将当前时间与/var/spool/anacron目录下面的文件里面的时间戳作对比，如果需要则去运行/etc/anacrontab对应的条目。这也是为什么/etc/anacrontab文件里面只定义了cron.daily、cron.weekly与cron.monthly，而没有定义cron.hourly，因为cron.daily、cron.weekly与cron.monthly其实是由cron.hourly调起来的。
每小时运行的0anacron只负责进行时间戳的比对，如果当前日期和上次运行anacron的日期不符，说明系统停机过了，就会启动anacron这
支程序，再由anacron根据/etc/anacrontab配置进一步判断，然后去运行cron.daily、cron.weekly与cron.monthly里面未完成的任务。

详细流程：[Anacron 排程服務](https://www.weithenn.org/2017/10/centos-74-journey-part11.html)

## crontab 注意问题

1. 注意环境变量问题

在crontab文件中定义多个调度任务时，需要特别注意的一个问题就是环境变量的设置

> ```
> # 脚本中涉及文件路径时写全局路径；
> # 脚本执行要用到java或其他环境变量时，通过source命令引入环境变量，如：
> cat` `start_cbp.sh
> #!/bin/sh
> source` `/etc/profile
> export` `RUN_CONF=``/home/d139/conf/platform/cbp/cbp_jboss``.conf
> /usr/local/jboss-4``.0.5``/bin/run``.sh -c mev &
> ```
>
>  
>
> ```
> # 当手动执行脚本OK，但是crontab死活不执行时。可以尝试在crontab中直接引入环境变量解决问题。
> 0 * * * * . ``/etc/profile``;``/bin/sh` `/var/www/java/audit_no_count/bin/restart_audit``.sh
> ```

2. 系统级任务调度与用户级任务调度

> ```
> root用户的任务调度操作可以通过“``crontab` `–uroot –e”来设置，也可以将调度任务直接写入``/etc/crontab``文件，需要注意的是，如果要定义一个定时重启系统的任务，就必须将任务放到``/etc/crontab``文件，即使在root用户下创建一个定时重启系统的任务也是无效的。
> ```

3.其他注意事项

> ```
> 当``crontab``突然失效时，可以尝试``/etc/init``.d``/crond` `restart解决问题。或者查看日志看某个job有没有执行/报错``tail` `-f ``/var/log/cron``。
> 千万别乱运行``crontab` `-r。它从Crontab目录（``/var/spool/cron``）中删除用户的Crontab文件。删除了该用户的所有``crontab``都没了。
> 在``crontab``中%是有特殊含义的，表示换行的意思。如果要用的话必须进行转义\%，如经常用的``date` `‘+%Y%m%d’在``crontab``里是不会执行的，应该换成``date` `‘+\%Y\%m\%d’
> ```

4. 生产调试定时任务

```
1.增加执行任务的频率调试``2.调整系统时间调试任务，提前5分钟  -->不用于生产环境``3.通过脚本日志输出调试定时 任务``4.注意一些任务命令带来的问题    -->确保命令的正确性
```

5.crontab箴言

```
1.环境变量问题，例如``crontab``不能识别Java的环境变量``  ``crontab``执行shell时，只能识别为数不多的环境变量，普通的环境变量是无法识别的，所以在编写shell时，最好使用``export``重新声明变量，确保脚本执行。 ``2.命令的执行最好用脚本``3.脚本权限加``/bin/sh``，规范路径``/server/scripts``4.时间变量用反斜线转义，最好用脚本``5.定时任务添加注释``6.>``/dev/null` `2>&1  ==>&>``/dev/null``,别随意打印日志文件``7.定时任务里面的程序脚本尽量用全路径``8.避免不必要的程序以及命令输出``9.定时任务之前添加注释``10.打包到文件目录的上一级
```

`参考地址:`

[crontab 教学](https://blog.gtwang.org/linux/linux-crontab-cron-job-tutorial-and-examples/)

[Linux crontab命令详解](https://www.cnblogs.com/ftl1012/p/crontab.html)

