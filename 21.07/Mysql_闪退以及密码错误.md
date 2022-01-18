# 1.闪退

## 用命令行进行开启服务

1. 以管理员的权限 net start mysql ，开启mysql服务
2. 以管理员的权限 net stop mysql ，关闭mysql服务
3. 以管理员的权限 mysqld -remove ，卸载mysql服务



# 2.密码错误

## 2.1mysql密码错误解决方法

1.确认sql目录下有没有data（Data）文件夹，如果有就删掉。(我的是解压版本的)

![img](http://img.baboo.fun/21.07/mysql_st_1.PNG)



2.然后在cmd输入mysqld --initialize，等待一段时间（这段时间就是在创建data（Data）文件夹），然后就再次输入net start mysql便可，mysql服务器会启动，这个之后 重新登录时会产生一个随机密码,先关闭cmd。



3.随后以管理员身份运行 cmd,我们先找到随机密码，可以搜索.err文件（找文件目录:**mysql\bin\data\ .err 文件 ，或寻找你的安装按目录），貌似里面就这一个.err的文件,用记事本打开，直接查找password，后面就是随机密码。

![img](http://img.baboo.fun/21.07/mysql_st_2.PNG)

## 2.2修改密码

1.以管理员身份打开cmd

2.进入数据库：mysql -u root -p

​	然后会提示输入密码,输入上面找到的密码。

3.数据库修改密码：ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '你的密码';  




