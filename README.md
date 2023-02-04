# linux_RT3070_STA

RT3070网卡，有官方驱动源码，尝试移植到S5P6818平台。

淘宝能买到带天线的。

![](https://files.catbox.moe/odlwkh.png)

![](https://files.catbox.moe/6tl45m.jpg)



### 1.编译源码

```shell
cd /home/alientek/rt3070_sta_s5p6818/
sudo make ARCH=arm KBUILD_NOPEDANTIC=1 
```

编译可能会出现的错误：

```shell
/home/gec/rt3070_sta/os/linux/../../os/linux/sta_ioctl.c:2227:2: error: unknown 
field 'private' specified in initializer
 .private = (iw_handler *) rt_priv_handlers,
```

这是由于内核中没有开启此宏

![](https://files.catbox.moe/kgdisf.png)

解决方案：
	在make menuconfig中配置选项。
	配置：

![](https://files.catbox.moe/exo6kx.png)

​	保存配置后重新编译内核，然后烧录内核



### 2.下载编译好的目标文件

编译生成目标文件rt2870.bin、rt3070sta.ko、RT2870STA.dat三个文件，后面需要将rt3070sta.ko和RT2870STA.dat这两个文件下载到板子上。

文件路径

```shell
./rt3070_sta_s5p6818/RT2870STA.dat
./rt3070_sta_s5p6818/os/linux/rt3070sta.ko
./common/rt2870.bin
```



### 3.加载驱动模块 rt3070sta.ko

```shell
cd /
tftp 192.168.22.xxx -gr rt3070sta.ko #把驱动文件拷贝到开发板
insmod rt3070sta.ko
#如果不加载驱动模块的话系统会无法识别插入的usb无线网卡
```



### 4.启动网卡

①首先插上RT3070网卡

②然后通过 ifconfig -a 命令查看是否有网卡ra0

![](https://files.catbox.moe/nm1thd.png)



③要将RT2870STA.dat文件拷贝到开发板的/etc/Wireless/RT2870STA目录下，否则无法启动网卡ra0

```shell
mkdir -p /etc/Wireless/RT2870STA
cd /etc/Wireless/RT2870STA/
tftp 192.168.22.xxx -gr RT2870STA.dat
```

④启动无线网卡ra0

```shell
ifconfig ra0 up 
```



### 5.配置WiFi名和WiFi密码

```shell
#将无线网卡的配置信息写入连接配置文件
wpa_passphrase tea 123456789 > /etc/wpa_supplicant.conf
 
cat /etc/wpa_supplicant.conf
#其中：
#xiaoming： 连接目标wifi的名字
#123456789：连接目标wifi的密码
```



### 6.为无线网卡ra0加载配置

```shell
wpa_supplicant -B -d -Dwext -ira0 -c /etc/wpa_supplicant.conf 
 
-d ：增加调试信息 
-B：后台守护进程 
-c：指定配置文件 
-Dwext：wext为驱动名称 
-ira0 ：ra0为网络接口名称
```

得到反馈信息：

![](https://files.catbox.moe/2or0a4.png)



### 7.动态获取IP

```shell
udhcpc -ira0
```

得到反馈信息：

![](https://files.catbox.moe/x1p413.png)



### 8.测试网络

```shell
ping www.baidu.com
```

得到反馈信息：

![](https://files.catbox.moe/qs5kff.png)

如果连不上外网，则在/etc/profile 添加 route add default gw 192.168.22.1 然后重启 //192.168.22.1写自己的网
关地址

如果DNS无法解析，则在输入命令 echo nameserver 8.8.8.8 > /etc/resolv.conf //8.8.8.8为谷歌的DNS服务器地址

