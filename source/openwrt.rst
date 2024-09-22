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