# 先重装系统为centos

## 开启可视化

https://blog.csdn.net/yh0503/article/details/86621623

 

## 去qt官网下载5.13.2版本

### centos 下载命令wget 

https://download.qt.io/archive/qt/

```bash
https://download.qt.io/archive/qt/5.13/5.13.2/qt-opensource-linux-x64-5.13.2.run
```



###  安装

https://blog.csdn.net/qq_40642828/article/details/104339628

- 对qt-opensource-linux-x64-5.14.1.run文件赋予可执行权限：

```bash
sudo chmod 777 qt-opensource-linux-x64-5.13.2.run
```

- 再在qt-opensource-linux-x64-5.14.1.run文件所在目录下运行.run文件

  ```bash
  ./qt-opensource-linux-x64-5.13.2.run
  ```



### 环境配置

ubuntu 中的环境变量配置文件为 /etc/profile , 打开此文件

```
sudo vim /etc/profile
```


在底行模式下按 G 进入最后一行，点击 i 进入 编辑模式，添加下面语句（以自己实际安装地址为主）

```
export PATH="/opt/qt/5.14.1/Tools/QtCreator/bin:$PATH"
export PATH="/opt/qt/5.14.1/gcc_64/bin:$PATH"
```

 

## 安装完qt发现缺少opengl的依赖：

https://blog.csdn.net/wwkaven/article/details/37755259

![img](https://baboo.obs.cn-east-3.myhuaweicloud.com/21.07/1_1.jpg)

![img](http://img.baboo.fun/21.07/1_1.jpg)

<img src="http://img.baboo.fun/21.07/1_1.jpg">

```bash
sudo apt-get install libgl1-mesa-dev 
greeter-show-manual-login=true
```



### 运行捕鱼达人游戏的服务端

注意防火墙端口是否已经打开