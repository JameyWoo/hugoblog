




在 wsl 里 安装 mysql5.7, 设置密码的时候折腾了半天, 看了几十个教程, 都没用, 最后终于看到了一篇, 解决了问题, 特此记录. 

[最后帮我解决问题的是这篇博客](https://blog.csdn.net/qq_38737992/article/details/81090373). **根据我的情况有所调整.** 



## 设置无密码

首先尝试

```
mysql -u root -p
```

进入数据库, 无密码, 无法进入

于是设置无密码进入数据库方式, 执行

```
sudo vim /etc/mysql/mysql.conf.d/mysqld.conf
```

在 `[mysqld]` 一栏下方添加

```
skip-grant-tables
```

然后执行

```
sudo service mysql restart
```

重启数据库

即可无密码进入数据库



## 修改密码

再执行

```
mysql -u root -p
```

需要输入密码时直接回车

然后执行

**注意: your_password 字段设置成自己的密码**

```
use mysql;

update mysql.user set authentication_string=password('your_password') where user='root' and Host ='localhost';

update mysql.user set plugin="mysql_native_password";


flush privileges;

quit;
```

我看的很多其他博客很类似, 但时没有 `update mysql.user set plugin="mysql_native_password";` 这一句. 个人觉得这一句非常关键.



然后回到

```
sudo vim /etc/mysql/mysql.conf.d/mysqld.conf
```

将之前设置的 `skip-grant-tables` 取消.

再

```
mysql -u root -p
```

输入密码, 即可进入数据库.

