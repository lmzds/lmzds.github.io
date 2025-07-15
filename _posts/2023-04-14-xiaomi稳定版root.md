---
title: 小米手机稳定版root
date: 2023-04-14 19:00 +0800
categories: [SRC,MiFlash]
tags: [稳定版,root]
media_subpath: /
---

小米手机在原生稳定版的情况下，获取root权限。

可能会变砖！！！自行决定！



# 一，打开USB调试

1. 打开开发者选项：设置-我的设备-全部参数-MIUI版本，连续点击，直到提示已打开开发者选项

2. 打开USB调试：设置-更多设置-开发者选项-USB调试



# 二，获取当前ROM版本的boot.img文件

1. 获取当前ROM的完整卡刷包，https://xiaomirom.com/rom/mi-6x-wayne-china-fastboot-recovery-rom/   这里查找，或者通过手机下载
2. 在ROM压缩包里找到boot.img文件，提取出来，放到手机的download文件夹。

![image-20230414185604632](assets/image-20230414185604632.png)



# 三，获取Magisk adb等文件

链接：https://pan.baidu.com/s/1aEREgjcIeC-y1_0oiImFcg?pwd=k8xx 
提取码：k8xx

![image-20230414190103447](assets/image-20230414190103447.png)

# 四，破解boot.img文件



![image-20230414190500402](assets/image-20230414190500402.png)

方式选择 选择并修补一个文件

**![image-20230414190639950](assets/image-20230414190639950.png)**

把文件放到电脑上，工具的目录下面。



# 五，刷入img

连接手机，adb调式开启

第一条命令可能要配对，同意一下就好，之后再打命令。

```shell
.\adb.exe reboot bootloader
.\fastboot.exe flash boot .\magisk_patched-26100_pBgEM.img
```

![image-20230414190929423](assets/image-20230414190929423.png)

重启后，选择直接安装。安装完重启后，就完成了。

![image-20230414191206027](assets/image-20230414191206027.png)

![image-20230414191425516](assets/image-20230414191425516.png)

![image-20230414191458878](assets/image-20230414191458878.png)