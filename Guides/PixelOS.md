#记一次给小米mix编译PixelOS的过程
偶然发现了这个[PixelOS](https://pixelos.net)，[GitHub](https://github.com/PixelOS-Pixelish)
![PixelOS](https://raw.githubusercontent.com/PixelOS-Pixelish/official_devices/thirteen/banners/PixelOS-11-Sept-2022.png)
##编译需要
**Linux**系统，本人使用Ubuntu22.04LTS
**16GB**以上内存，本人16GB，只能编译安卓11以下版本，再往上系统就会崩溃
**300GB**以上存储，如果想同时存储多个ROM源代码则需要更多（每个ROM源代码少说100多GB）

##配置编译环境
###安装 platform-tools
可以从[Google](dl.google.com/android/repository/platform-tools-latest-linux.zip)下载， 然后解压：
```
unzip platform-tools-latest-linux.zip -d ~
```
> 提示: 文件的名称可能与此命令中的名称不同，需要做相应调整。

现在需要将adb和fastboot添加到环境变量PATH中。打开**~/.profile**并添加如下内容：
```
# add Android SDK platform tools to path
if [ -d "$HOME/platform-tools" ] ; then
    PATH="$HOME/platform-tools:$PATH"
fi
```
然后运行**source ~/.profile**来更新环境变量

###安装编译所需的软件包
在终端中，一次性输入以下命令：
```
sudo apt install -y bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev libelf-dev liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev
```
**耐心等待**
如果 Ubuntu 版本低于 20.04 (focal), 还需要加上：
libwxgtk3.0-dev
如果 Ubuntu 版本低于 16.04 (xenial), 还需要再加上：
libwxgtk2.8-dev

###Python
对于没有安装Python的用户
终端输入：
```
sudo apt install -y python3
sudo ln -s /usr/bin/python3 /usr/bin/python
```
使用这个命令查看python版本：
```
python --version
```

###新建文件夹
```
mkdir -p ~/bin
mkdir -p ~/android/pixel
```
没有任何输出即为成功

###安装repo命令
运行以下命令来下载repo二进制文件，并使其可执行：
```
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

###将~/bin加入到环境变量
在最新版本的Ubuntu中，~/bin应该已经被添加到环境变量。
你可以使用文本编辑器打开~/.profile并验证以下代码是否存在来检查这一点（如果没被添加，请手动添加）：
```
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi
```
然后运行**source ~/.profile**来更新环境变量

###配置 git
```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```
没有任何输出即为成功

##同步源代码
###初始化存储库
输入以下内容以初始化存储库：

```
cd ~/android/pixel
repo init -u https://github.com/PixelOS-Pixelish/manifest -b eleven
```
注意：如果出现类似以下提示：
/usr/bin/env: “python”: 没有那个文件或目录
那么需要安装Python，方法见上文

###下载源代码
输入以下命令来同步源码:
```
repo sync -j4
```
-j4 是指使用四线程同步源码
如果你在同步时遇到问题，可以尝试降低线程。

备注: 这可能需要亿点时间，具体取决你于的网速，我用了一个上午。在此期间去喝杯啤酒/咖啡/茶/小睡吧！
> 提示: repo sync是用来从PixelOS和Google获取最新的源码。 如果你想更新源码，运行它就好了。注意，如果你对源码进行了更改，则repo sync可能会清空你的更改！

出现：
repo sync has finished successfully.
即为成功
如果失败的话就重复执行上面的命令指直到成功

##配置设备专用代码
由于PixelOS和PE、AOSP以及ppui的设备树通用，所以这里直接使用PE的设备树
[PE的设备源代码](https://github.com/PixelExperience-Xiaomi-MSM8996)
**注意！！！以下操作请在源码根目录下完成！！！**
###设备树
使用[PE的设备树](https://github.com/PixelExperience-Xiaomi-MSM8996/device_xiaomi_lithium)
```
git clone https://github.com/PixelExperience-Xiaomi-MSM8996/device_xiaomi_lithium.git -b eleven device/xiaomi/lithium
```

**但是，这个设备树有问题，如果直接使用那么编译出来的ROM是无法显示导航键的！！！**
那么如何解决呢？很简单，找到设备树（device/xiaomi/lithium）并打开这个路径overlay/frameworks/base/core/res/res/values
进入后找到config.xml并使用文本编辑器打开，在倒数第二行按一下回车并输入：
```
<bool name="config_showNavigationBar">true</bool>
```
**注意空格！！！**
###通用设备树
使用[PE的通用设备树](https://github.com/PixelExperience-Xiaomi-MSM8996/device_xiaomi_msm8996-common)
```
git clone https://github.com/PixelExperience-Xiaomi-MSM8996/device_xiaomi_msm8996-common.git -b eleven device/xiaomi/msm8996-common
```
###内核
使用[LineageOS内核](https://github.com/LineageOS/android_kernel_xiaomi_msm8996)
```
git clone https://github.com/LineageOS/android_kernel_xiaomi_msm8996.git kernel/xiaomi/msm8996
```
###vendor
使用[PE的vendor](https://github.com/PixelExperience-Xiaomi-MSM8996/proprietary_vendor_xiaomi)
```
git clone https://github.com/PixelExperience-Xiaomi-MSM8996/proprietary_vendor_xiaomi.git -b eleven vendor/xiaomi
```
##Hardware
```
git clone https://github.com/LineageOS/android_hardware_xiaomi.git -b lineage-18.1 hardware/xiaomi
```
##编译
```
. build/envsetup.sh
lunch aosp_lithium-user
mka bacon
```
到这里就基本结束了，说一下如果出现报错该怎么办
首先，[Google](https://google.com)
如果没有ti或没有答案的话也可以在本贴下回复，来说一下如何回复
推荐的格式：xx系统，在步骤x中执行x命令时出现了x问题，（可选）在尝试了x方法后依然这样或显示y错误（如果有log就更好了）
如：Ubuntu22.04LTS，在同步源代码中执行repo sync -j4命令时出现了x问题，在尝试了更改线程后依然这样，log：[log文件链接]（这一般是由于网络不好导致的，可以使用ti）
log位置：在out目录下有error.log等类似文件，请上传到**[蓝奏云](https://lanzou.com)（推荐）或其他不强制下载app或强制登陆的网盘，GitHub也可以，拒绝百度盘！！！**

