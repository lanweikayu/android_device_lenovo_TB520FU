# 联想 Yoga Tab Plus 12.7（TB520FU）的 TWRP 设备树

[English](README.md)

适用于联想 Yoga Tab Plus 12.7、型号 `TB520FU` 的 TWRP 设备树。

- 维护者：兰微卡鱼（酷安 @兰微卡鱼，GitHub [@lanweikayu](https://github.com/lanweikayu)）
- 仓库地址：<https://github.com/lanweikayu/android_device_lenovo_TB520FU>
- 问题反馈：<https://github.com/lanweikayu/android_device_lenovo_TB520FU/issues>
- 预编译镜像：<https://github.com/lanweikayu/android_device_lenovo_TB520FU/releases>

## 参考项目

本设备树基于联想 Legion Y700（TB321FU）的原始 TWRP 设备树制作：

- [联想 TB321FU 的 TWRP](https://github.com/polygraphene/android_device_lenovo_TB321FU)


## 自行构建

### 环境要求

- 64 位 Linux 构建环境
- Git 和 Android `repo` 工具
- 编译 AOSP 所需的常规依赖包
- 足够存放 Android 16 源码及编译输出的磁盘空间

### 同步源码

新建工作目录并同步未经修改的 TWRP-Test Android 16 官方源码：

```shell
mkdir -p twrp
cd twrp
repo init --depth=1 -u https://github.com/TWRP-Test/platform_manifest_twrp_aosp.git -b twrp-16.0
repo sync
```

不要将 `bootable/recovery` 替换为设备专用 fork。

### 添加设备树

将本仓库克隆到 TWRP 源码中的以下准确路径：

```text
device/lenovo/TB520FU
```

```shell
mkdir -p device/lenovo
git clone https://github.com/lanweikayu/android_device_lenovo_TB520FU.git device/lenovo/TB520FU
```

完成后应能找到以下产品配置文件：

```text
device/lenovo/TB520FU/AndroidProducts.mk
device/lenovo/TB520FU/twrp_TB520FU.mk
```

### 准备编译环境

在已同步的 TWRP 源码根目录中执行：

```shell
export ALLOW_MISSING_DEPENDENCIES=true
source build/envsetup.sh
lunch twrp_TB520FU-bp2a-eng
```

lunch 命令必须包含 `bp2a` release 配置。Android 16 源码不接受较短的
`lunch twrp_TB520FU-eng` 目标。

### 构建横屏版

横屏是默认配置。先清除竖屏变量，再清理上一个变体的安装输出并构建：

```shell
unset USE_PORTRAIT
lunch twrp_TB520FU-bp2a-eng
mka installclean
mka recoveryimage
```

生成的镜像位于：

```text
out/target/product/TB520FU/recovery.img
```

构建另一变体前，应先将横屏镜像保存到 `out/target/product` 之外：

```shell
mkdir -p builds
cp out/target/product/TB520FU/recovery.img \
    "builds/TWRP-3.7.1_16-TB520FU-landscape.img"
```

### 构建竖屏版

竖屏版使用 `portrait_hdpi`、90 度旋转和适配 TB520FU 面板的竖屏布局。
设置 `USE_PORTRAIT=true`，重新载入产品配置，清理安装输出后构建：

```shell
export USE_PORTRAIT=true
lunch twrp_TB520FU-bp2a-eng
mka installclean
mka recoveryimage USE_PORTRAIT=true
```

保存竖屏镜像并清除环境变量：

```shell
cp out/target/product/TB520FU/recovery.img \
    "builds/TWRP-3.7.1_16-TB520FU-portrait.img"
unset USE_PORTRAIT
```

两个变体使用相同的输出路径，因此未提前复制横屏镜像时，竖屏构建会将其覆盖。
`mka installclean` 只清理已安装的产品文件并保留编译对象缓存，适合在两个主题间切换。

### 验证镜像

每次构建完成后，在重命名或刷入镜像之前检查大小和 AVB 元数据：

```shell
stat --format='%s bytes' out/target/product/TB520FU/recovery.img
sha256sum out/target/product/TB520FU/recovery.img
python3 external/avb/avbtool.py verify_image \
    --image out/target/product/TB520FU/recovery.img \
    --key external/avb/test/data/testkey_rsa4096.pem
```

镜像大小必须正好为 `104857600` 字节。成功的编译会以
`build completed successfully` 结束，并在上述路径生成已签名镜像。

## 预编译镜像

横屏版和竖屏版 Recovery 镜像均发布在本仓库的
[Releases](https://github.com/lanweikayu/android_device_lenovo_TB520FU/releases) 页面：

```text
TWRP-3.7.1_16-TB520FU-landscape.img   横屏版（默认），104857600 字节
TWRP-3.7.1_16-TB520FU-portrait.img    竖屏版，104857600 字节
```

两个镜像都包含 AVB 元数据（使用 TWRP 测试密钥签名）。下载后可对照 Release
附带的 `SHA256SUMS.txt` 校验文件完整性。

## 功能状态

移植状态（设备验证中）：

- [X] ADB
- [X] 备份和恢复
- [X] 数据解密（metadata/FBE）
- [X] 屏幕显示
- [X] 镜像刷入
- [X] 分区挂载
- [X] 触控
- [X] Fastbootd
- [X] USB OTG


## 设备信息

- 平台：Qualcomm `pineapple` / SM8650-AB Snapdragon 8 Gen 3
- GPU：Adreno 750
- 内核：arm64，4 KiB 页大小
- 屏幕：12.7 英寸 LTPS 2944x1840（3K），144 Hz，400 dpi
- 分区：Virtual A/B，具有独立的 `recovery_a` 和 `recovery_b`
- Recovery 分区大小：104857600 字节
- microSD 卡槽：无

## 刷入方法

刷入前请为两个原厂 Recovery 槽位保留经过校验的备份。Recovery 镜像无法启动时，
可能只能通过 Qualcomm 9008（EDL）模式恢复设备。

### 通过 bootloader 刷入

电脑端应使用较新的 Android platform-tools。

1. 从 Android 或可正常工作的 Recovery 进入 bootloader：

```shell
adb reboot bootloader
```

2. 确认电脑识别设备：

```shell
fastboot devices
```

3. 选择横屏或竖屏镜像并刷入两个 Recovery 槽位：

```shell
RECOVERY_IMAGE="builds/TWRP-3.7.1_16-TB520FU-landscape.img"
fastboot flash recovery_a "$RECOVERY_IMAGE"
fastboot flash recovery_b "$RECOVERY_IMAGE"
fastboot reboot recovery
```

需要竖屏版时替换为竖屏镜像文件名。也可以只刷写一个槽位，此时另一个槽位会保留
原有 Recovery。

## 致谢

- [android_device_lenovo_TB321FU](https://github.com/polygraphene/android_device_lenovo_TB321FU) - 本项目所基于的联想 TB321FU 原始设备树。
- [TWRP-Test](https://github.com/TWRP-Test/platform_manifest_twrp_aosp)
- [TWRP](https://twrp.me/)
