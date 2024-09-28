MT7621A移植openwrt 23.05.3
==============================

1. tools/elfutils/Makefile增加  ``--disable-demangler``编译选项
2. target/linux/ramips/dts/mt7621_mqmaker_witi.dts添加
````
  chosen {
    bootargs = "console=ttyS0,115200";
  };
````
实现串口波特率为115200
3. 选择了 ``odhcpd``就不要选择 ``odhcpd-ipv6only``
4. 目标平台选择Target System= ``MediaTek Ralink MIPS``, Subtarget= ``MT7621 base boards``, Target Profile= ``MQmaker WITI``。
详见文件 
.. include:: mt7621_mqmaker_witi.config
5. OpenWrt编译选项
  执行make menuconfig，报错：
```
touch: cannot touch '/opt/mipsel-linux-gcc/host/.prereq-build': No such file or directory
make: *** [/mex/openwrt-sdk-ramips-mt7621/include/toplevel.mk:183: /opt/mipsel-linux-gcc/host/.prereq-build] Error 1
```
  导致编译报错的原因是权限问题，执行sudo make menuconfig可解决。
6. 编译内核模块
  解压sdk文件，目录中有package/kernel，主要存放内核模块。执行sudo make package/hello/compile V=sc进行编译
  出compile命令外还有clean、install等命令，不同的是需要在hello/Makefile中添加相应的脚本
7. 海创科技的主板网口布局从左往右[lan1 lan2 lan3 lan4 eth0]
8. openwrt如何支持vsftpd
  - 创建/home/ftp
  - ftp:*:0:0:ftp:/home/ftp:/bin/false 设置ftp账户权限
  - ftp:tTCk7479EdgKk:0:0:99999:7::: 设置ftp账号为空格， ``tTCk7479EdgKk``是使用openssl passwd命令生成
