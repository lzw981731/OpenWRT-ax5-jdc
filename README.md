# OpenWRT-ax5-jdc
基于 OpenWRT-CI 云编译的 OpenWRT 固件，支持 DAED 内核级透明代理及各类科学上网插件。

## 适用设备
**此固件专供 Redmi AX5 JDC (京东云版) 使用。**

---

## 固件版本说明

### 📶 1. 带WiFi版本 (`IPQ60XX-WIFI`)
此固件仅给ax5 jdc 使用。自带开源硬件加速！

- **源码**：https://github.com/VIKINGYFY/immortalwrt.git
- **分支**：main
- **提交**：6bb76ef
- **配置**：IPQ60XX-WIFI
- **平台**：qualcommax
- **登录地址**：`192.168.1.2`
- **登录密码**：无
- **WIFI名称**：`Vipkj-WRT`
- **WIFI密码**：`12345678`
- **内核版本**：6.18.26
- **插件列表**：luci-app-autoreboot, luci-app-cpufreq, luci-app-dae, luci-app-daed, luci-app-ddns-go, luci-app-firewall, luci-app-gecoosac, luci-app-package-manager, luci-app-passwall, luci-app-passwall2, luci-app-sqm, luci-app-ttyd, luci-app-upnp, luci-app-vlmcsd, luci-app-wolplus, luci-app-zerotier, luci-theme-argon

### 🔌 2. 不带WiFi版本 (`IPQ60XX-NOWIFI`)
此固件仅给ax5 jdc 使用，且无wifi。自带开源硬件加速！

- **源码**：https://github.com/VIKINGYFY/immortalwrt.git
- **分支**：main
- **提交**：6bb76ef
- **配置**：IPQ60XX-NOWIFI
- **平台**：qualcommax
- **登录地址**：`192.168.1.2`
- **登录密码**：无
- **WIFI名称**：无
- **WIFI密码**：无
- **内核版本**：6.18.26
- **插件列表**：luci-app-autoreboot, luci-app-cpufreq, luci-app-dae, luci-app-daed, luci-app-ddns-go, luci-app-firewall, luci-app-gecoosac, luci-app-package-manager, luci-app-passwall, luci-app-passwall2, luci-app-sqm, luci-app-ttyd, luci-app-upnp, luci-app-vlmcsd, luci-app-wolplus, luci-app-zerotier, luci-theme-argon

---

## 🛠️ 刷机指南

### 第一步：刷入 U-Boot 和 分区表
所需文件已上传至本仓库 [`u-boot` 目录](https://github.com/lzw981731/OpenWRT-ax5-jdc/tree/main/u-boot) 中。
1. 按照相关教程刷入大雕的 `u-boot.bin`。
2. 在大雕的后台更新适配本固件的 u-boot（使用本仓库提供的 `u-boot.bin` 文件）。
3. 断电按住 reset 按键，进入 uboot 界面：[http://192.168.1.1/img.html](http://192.168.1.1/img.html)
4. 刷入 AX5 JDC 版的分区表文件：`gpt_ax5_jdc_HLOS_12M_ROOTFS_1G.bin`。

### 第二步：刷入固件
1. 刷完分区表后，断电按住 reset 按键再次进入 uboot，访问：[http://192.168.1.1/big.html](http://192.168.1.1/big.html)
2. 刷入带 `factory` 字样的固件（此地址专用于刷入 12M 内核的 factory 固件，界面会带有 *firmware which has a 12M kernel* 的字眼）。
   - `WIFI-YES` 代表带WIFI版本。
   - `WIFI-NO` 代表无WIFI版。
   - 固件每周编译一次。
3. 以后更新固件时，可以直接在 OpenWrt 后台刷入 `sysupgrade` 固件。

---

## ⚠️ 特别注意：备份与恢复 ART 分区

> **危险提示**：如果你是从 QWRT 刷到开源固件，**请务必先备份好 ART 分区**！从 QWRT 刷到开源固件极有可能造成 ART 分区损坏，导致无线功能完全失效。

### 备份 ART 分区

1. 在路由器终端执行命令查找分区名：
```bash
grep PARTNAME /sys/block/mmcblk0/mmcblk0p*/uevent
```
执行后，你会看到类似这样的输出，找到 `0:ART` 对应的分区号（例如 `mmcblk0p15`）：
```text
/sys/block/mmcblk0/mmcblk0p1/uevent:PARTNAME=0:SBL1
/sys/block/mmcblk0/mmcblk0p2/uevent:PARTNAME=0:MIBIB
...
/sys/block/mmcblk0/mmcblk0p15/uevent:PARTNAME=0:ART
```

2. 备份该分区：
```bash
dd if=/dev/mmcblk0p15 of=/tmp/art_backup.bin
```

### 验证并恢复 ART 分区

1. **验证备份文件大小**：
```bash
ls -l /tmp/art_backup.bin
```
> **注意**：仔细看输出结果中的文件大小字段。它必须是精确的 **262144** 字节（即 256KB）。如果多一个字节或少一个字节，说明文件损坏，**立刻停止操作，重新生成！**

2. **执行恢复（写入 eMMC）**：
确认文件大小无误后，使用 `dd` 命令将文件反向写入到我们之前查出的块设备分区中：
```bash
dd if=/tmp/art_backup.bin of=/dev/mmcblk0p15
```
执行后，终端会很快返回类似 `512+0 records in / 512+0 records out` 的提示，表示写入完成。

3. **强制同步并重启**：
为了确保数据从缓存完全写入到 eMMC 闪存芯片中，请务必执行一次同步命令：
```bash
sync
```
最后，重启路由器让底层硬件重新加载无线校准数据：
```bash
reboot
```
