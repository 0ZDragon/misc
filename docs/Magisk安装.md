# 安装

如果您已经安装了 Magisk，那么**强烈建议**您使用“直接安装（推荐）”方法通过 Magisk 应用程序直接升级。以下教程仅针对初始安装。
[原链接](https://github.com/topjohnwu/Magisk/blob/master/docs/install.md)

## 入门

在你开始之前：

- 本教程假定您了解如何使用 `adb` 和 `fastboot`
- 如果您还计划安装自定义内核，请在 Magisk 之后安装它
- 您设备的BL必须解锁

---

下载并安装最新的 [Magisk app](https://github.com/topjohnwu/Magisk/releases/latest) 应用程序。在主屏幕中，您应该看到：

<p align="center"><img src="1.png" width="500"/></p>

**Ramdisk** 的结果决定了您的设备的boot分区中是否有 ramdisk。 如果您的设备没有启用 ramdisk，请在继续之前阅读[REC中的 Magisk](#REC中的 Magisk)。

> _（不幸的是，有些设备的引导加载程序会接受 ramdisk，即使它不应该接受。 在这种情况下，您必须按照说明进行操作，就好像您的设备的引导分区确实包含 ramdisk 一样。 没有办法检测到这一点，因此唯一可以确定的方法就是实际尝试。 幸运的是，据我们所知，只有部分小米设备具有此属性，所以大多数人可以简单地忽略这条信息。）_

如果您使用的是搭载 Android 9.0 或更高版本的三星设备，您现在可以跳转到[它自己的部分](#三星-system-as-root)。

如果您的设备有启用 ramdisk，请获取 `boot.img` 的副本。
如果您的设备**没有**启用 ramdisk，请获取 `recovery.img` 的副本。
您应该能够从官方固件包或自定义 ROM 刷机包中提取所需的文件。

接下来，我们需要知道您的设备是否有单独的 `vbmeta` 分区。

- 如果您的官方固件包中包含 `vbmeta`，那么是的，您的设备**有**一个单独的 `vbmeta` 分区
- 您还可以通过将设备连接到 PC 并运行命令来进行检查：<br>
  `adb shell ls -l /dev/block/by-name`
- 如果您找到 `vbmeta`、`vbmeta_a` 或 `vbmeta_b`，那么是的，您的设备**有**一个单独的 `vbmeta` 分区
- 否则，您的设备**没有**单独的 `vbmeta` 分区。

快速回顾一下，此时，您应该已经知道并准备好了：

1. 您的设备是否有启用 ramdisk
2. 您的设备是否有单独的 `vbmeta` 分区
3. 基于 (1) 的 `boot.img` 或 `recovery.img`

让我们继续 [修补镜像](#修补镜像).

## 修补镜像

- 将 boot/recovery 镜像复制到您的设备
- 点击 Magisk 主屏幕中的**安装**按钮
- 如果您正在修补 recovery 镜像，请选中**“Recovery 模式”**选项
- 如果您的设备**没有**单独的 `vbmeta` 分区，请选中**“修补 boot 镜像中的 vbmeta”**选项
- 在方式中选择 **"选择并修补一个文件"** ，选择 boot/recovery 镜像
- 开始安装，并使用 adb 将修补后的镜像复制到您的 PC：<br>
  `adb pull /sdcard/Download/magisk_patched_[random_strings].img`
  **不要使用 MTP**，因为它会损坏大文件。
- 将修补后的 boot/recovery 镜像刷入到您的设备。<br>
  对于大多数设备，重启进入fastboot模式并使用命令刷入：<br>
  `fastboot flash boot /path/to/magisk_patched.img` or <br>
  `fastboot flash recovery /path/to/magisk_patched.img`
- （可选）如果您的设备有单独的 `vbmeta` 分区，您可以使用以下命令修补 `vbmeta` 分区：<br>
  `fastboot flash vbmeta --disable-verity --disable-verification vbmeta.img`
- 重新启动，瞧！

## 卸载

卸载 Magisk 的最简单方法是直接通过 Magisk 应用程序。 如果您坚持使用REC，请将 Magisk APK 重命名为 `uninstall.zip` 并像任何其他普通的刷机包一样刷新它。

## Recovery 中的 Magisk

如果您的设备在启动映像中没有 ramdisk，Magisk 别无选择，只能占据recovery分区。 对于这些设备，每次使用 Magisk 时都必须重新启动到REC

当 Magisk 占据 recovery 分区时，有一种特殊的机制可以让您 _实际_ 启动进入恢复模式。每个设备都有自己的启动到 recovery 模式按键组合，例如几乎所有的小米设备均为「电源」+「音量增大」、Galaxy S10（「电源」+「Bixby」+「音量增大」）。一旦您按下组合键并且设备出现启动画面，请松开所有按钮以启动进入 Magisk。如果您决定启动进入实际的 recovery 模式，请**长按音量增大按键直到看到恢复屏幕**。

作为总结，在 recovery 中安装 Magisk 后**（从关机开始）**：

- **（正常开机） → （没有Magisk的系统）**
- **（按键组合） → （启动画面） → （释放所有按钮） → （带 Magisk 的系统）**
- **（按键组合） → （启动画面） → （长按音量增大） → （Recovery 模式）**

（注意：在这种情况下，您**不能**使用自定义恢复来安装或升级 Magisk！！）

## 三星 （system-as-root）

> 如果您的三星设备未运行 Android 9.0 或更高版本，则您阅读的部分有误。

### 安装 Magisk 之前

- 安装 Magisk **会**触发 KNOX
- 第一次安装 Magisk **需要**完全擦除数据（这**不**包括解锁引导加载程序时的数据擦除）。请在继续之前备份您的数据！
- 下载支持您的设备的 Odin（仅在 Windows 上运行）。

### 解锁 BL

在较新的三星设备上解锁引导加载程序有一些注意事项。新引入的 `VaultKeeper` 服务将使BL在某些情况下拒绝任何非官方分区。

- 允许解锁BL：在**开发者选项中 → OEM 解锁**
- 重启到下载模式：将您的设备关机并按下您设备的下载模式组合键
- 长按音量增大解锁引导加载程序。**这将擦除您的数据并自动重启**。
- 完成初始设置。 跳过所有步骤，因为数据将在后续步骤中再次擦除。**在设置过程中将设备连接到互联网**。
- 开启开发者选项，**确认OEM解锁选项存在且为灰色**。这意味着 `VaultKeeper` 服务已经释放了BL。
- 您的BL现在在下载模式下接受非官方镜像

### 指示

- 使用 [samfirm.js](https://github.com/jesec/samfirm.js), [Frija](https://forum.xda-developers.com/s10-plus/how-to/tool-frija-samsung-firmware-downloader-t3910594), 或 [Samloader](https://forum.xda-developers.com/s10-plus/how-to/tool-samloader-samfirm-frija-replacement-t4105929) 直接从三星服务器下载设备的最新固件 zip。
- 解压缩固件并将 `AP` tar 文件复制到您的设备。 它通常命名为 `AP_[device_model_sw_ver].tar.md5`
- 点击 Magisk 主屏幕中的**安装**按钮
- 如果您的设备**没有**启用 ramdisk，请选中**“Recovery 模式”**选项
- 在方式中选择 **"选择并修补一个文件"** ，选择 `AP` tar 文件
- 开始安装，并使用 adb 将修补后的镜像复制到您的 PC：<br>
  `adb pull /sdcard/Download/magisk_patched_[random_strings].tar`<br>
  **不要使用 MTP**，因为它会损坏大文件。
- 重新启动到下载模式。 在您的 PC 上打开 Odin，将 `magisk_patched.tar` 作为 `AP`，连同原始固件中的 `BL`、`CP` 和 `CSC`（**不是** `HOME_CSC`，因为我们要**擦除数据**）一起刷入。
- 一旦 Odin 完成刷机，您的设备应该会自动重启。 如果被要求恢复出厂设置，请同意。
- 如果您的设备**没有**启用 ramdisk，请立即重新启动到 recovery 以启用 Magisk（原因在 Recovery 中的 Magisk 中说明）。
- 安装您已经下载的 [Magisk 应用程序](https://github.com/topjohnwu/Magisk/releases/latest) 并启动该应用程序。 它应该显示一个对话框，要求进行额外的设置。
- 让应用程序完成它的工作并自动重启设备。瞧！

### 系统升级

一旦你的三星设备获得了 root 权限，你就不能再通过 OTA 升级你的 Android 系统。 要升级设备的系统，您必须手动下载新的固件 zip 文件并完成上一节中编写的相同 `AP` 修补过程。**这里唯一的区别在于 Odin 刷机步骤：不要使用 `CSC` tar，而是使用 `HOME_CSC` tar，因为我们正在执行升级，而不是初始安装**。

### 注意事项

- **永远、永远不要**尝试将 `boot`、`recovery`或 `vbmeta` 分区恢复到原样！ 您这样做会破坏您的设备，并且从中恢复的唯一方法是清除数据并进行完整的 Odin 恢复。
- 要使用新固件升级您的设备，**切勿**出于上述原因直接使用原厂 `AP` tar 文件。 **始终**在 Magisk 应用程序中修补 `AP` 并改用它。
- 永远不要只刷 `AP`，否则 Odin 可能会缩小 `/data` 文件系统的大小。 升级时刷 `AP` + `BL` + `CP` + `HOME_CSC`。

## 第三方 Recovery

> **这种安装方法已被弃用，维护工作量很小。 你被警告了！**

仅当您的设备具有启用 ramdisk 时，才能使用第三方 Recovery进行安装。不再建议在新设备上通过第三方 Recovery安装 Magisk。 如果您遇到任何问题，请使用正确的[修补镜像](#修补镜像)方法。

- 下载 Magisk APK
- 将 `.apk` 文件扩展名重命名为 `.zip`，例如： `Magisk-v24.0.apk` → `Magisk-v24.0.zip`。如果您在重命名文件扩展名时遇到问题（例如在 Windows 上），请使用 Android 上的文件管理器或 TWRP 中包含的文件管理器来重命名文件。
- 像任何其他普通的刷机包一样刷入。
- 重新启动并检查是否安装了 Magisk 应用程序。 如果未自动安装，请手动安装 APK。

> 警告：模块的 `sepolicy.rule` 文件可能存储在 `cache` 分区中。不要清除 `CACHE` 分区！