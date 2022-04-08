# LINUX_RK356X_MTP

## 1.确认sdk版本。

```
ly@ubuntu101:~/rk_sdk/rk356x$ ls -al  .repo/manifests/rk356x_linux_release.xml
lrwxrwxrwx 1 ly ly 53 Feb 25 09:46 .repo/manifests/rk356x_linux_release.xml -> rk356x_linux/rk356x_linux_release_v1.2.3_20220108.xml
```



## 2.配置MTP功能。

### 2.1kernel 修改。

由于4.19 kernel 移除了f_mtp驱动，但是3568在4.19内核ffs上层并没有做好,所以直接使用4.4的驱动移植到4.19所以要移植驱动到kernel，并增加kernel config。

从kernel4.4 移植的文件如下，补丁见0001-usb-gadget-f_mtp-add-function-mtp。

```
# Changes to be committed:
#       modified:   arch/arm64/configs/rockchip_linux_defconfig
#       modified:   drivers/usb/gadget/Kconfig
#       modified:   drivers/usb/gadget/function/Makefile
#       new file:   drivers/usb/gadget/function/f_mtp.c
#       new file:   include/linux/usb/f_mtp.h
#       new file:   include/uapi/linux/usb/f_mtp.h
```

### 2.2 buildroot修改。

补丁见0001-config-add-usb-mtp-function。

```
diff --git a/configs/rockchip_rk3568_defconfig b/configs/rockchip_rk3568_defconfig
index 6351a0f..e6218bf 100644
--- a/configs/rockchip_rk3568_defconfig
+++ b/configs/rockchip_rk3568_defconfig
@@ -23,3 +23,4 @@
 BR2_PACKAGE_RKWIFIBT_AP6398S=y
 BR2_PACKAGE_RKWIFIBT_BTUART="ttyS8"
 BR2_PACKAGE_RKNPU2=y
+BR2_PACKAGE_MTP=y
```



## 3.测试mtp功能。

### 3.1 测试步骤。

```
[root@RK356X:/]# echo usb_mtp_en > /tmp/.usb_config
[root@RK356X:/]# cat /tmp/.usb_config
usb_mtp_en
[root@RK356X:/]# /etc/init.d/S50usbdevice restart
[ 1142.096928] android_work: sent uevent USB_STATE=DISCONNECTED
[ 1142.098233] mtp_release
[ 1142.177253] mtp_open
[root@RK356X:/]# [ 1144.437423] dwc3 fcc00000.dwc3: device reset
[ 1144.437614] android_work: did not send uevent (0 0           (null))
[ 1144.497562] android_work: sent uevent USB_STATE=CONNECTED
[ 1144.617890] configfs-gadget gadget: high-speed config #1: b
[ 1144.618368] android_work: sent uevent USB_STATE=CONFIGURED
```



### 3.2 测试结果。

![image-20220301150630264](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220301150630264.png)





## 4.特殊情况。

测试时编译buildroot mtp-server时，可能下载包会有很长时间，手动执行mtp-server，确认mtp-server有生效到板子。
