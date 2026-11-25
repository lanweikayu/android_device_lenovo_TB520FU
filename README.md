# TWRP device tree for Lenovo Yoga Tab Plus 12.7 (TB520FU)

[简体中文](README-zh-cn.md)

TWRP device tree for the Lenovo Yoga Tab Plus 12.7, model `TB520FU`.

- Maintainer: lanweikayu (Coolapk @兰微卡鱼, GitHub [@lanweikayu](https://github.com/lanweikayu))
- Repository: <https://github.com/lanweikayu/android_device_lenovo_TB520FU>
- Issues: <https://github.com/lanweikayu/android_device_lenovo_TB520FU/issues>
- Prebuilt images: <https://github.com/lanweikayu/android_device_lenovo_TB520FU/releases>

## References

This device tree is based on the original TWRP device tree for the Lenovo
Legion Y700 (TB321FU):

- [TWRP for Lenovo TB321FU](https://github.com/polygraphene/android_device_lenovo_TB321FU)


## Building

### Requirements

- A 64-bit Linux build environment
- Git and the Android `repo` tool
- The usual packages required to build AOSP
- Enough disk space for the Android 16 source tree and its build output

### Syncing the source

Create a working directory and sync the unmodified official TWRP-Test
Android 16 sources:

```shell
mkdir -p twrp
cd twrp
repo init --depth=1 -u https://github.com/TWRP-Test/platform_manifest_twrp_aosp.git -b twrp-16.0
repo sync
```

Do not replace `bootable/recovery` with a device-specific fork.

### Adding the device tree

Clone this repository to the exact path below inside the TWRP source tree:

```text
device/lenovo/TB520FU
```

```shell
mkdir -p device/lenovo
git clone https://github.com/lanweikayu/android_device_lenovo_TB520FU.git device/lenovo/TB520FU
```

Afterwards the following product configuration files must be present:

```text
device/lenovo/TB520FU/AndroidProducts.mk
device/lenovo/TB520FU/twrp_TB520FU.mk
```

### Setting up the build environment

From the root of the synced TWRP source tree:

```shell
export ALLOW_MISSING_DEPENDENCIES=true
source build/envsetup.sh
lunch twrp_TB520FU-bp2a-eng
```

The lunch command must include the `bp2a` release configuration. The Android 16
source tree does not accept the shorter `lunch twrp_TB520FU-eng` target.

### Building the landscape variant

Landscape is the default configuration. Clear the portrait variable, then clean
the installed output of the previous variant and build:

```shell
unset USE_PORTRAIT
lunch twrp_TB520FU-bp2a-eng
mka installclean
mka recoveryimage
```

The generated image is located at:

```text
out/target/product/TB520FU/recovery.img
```

Before building the other variant, save the landscape image outside
`out/target/product`:

```shell
mkdir -p builds
cp out/target/product/TB520FU/recovery.img \
    "builds/TWRP-3.7.1_16-TB520FU-landscape.img"
```

### Building the portrait variant

The portrait variant uses `portrait_hdpi`, 90 degree rotation and a portrait
layout matched to the TB520FU panel. Set `USE_PORTRAIT=true`, reload the product
configuration, clean the installed output and build:

```shell
export USE_PORTRAIT=true
lunch twrp_TB520FU-bp2a-eng
mka installclean
mka recoveryimage USE_PORTRAIT=true
```

Save the portrait image and clear the variable:

```shell
cp out/target/product/TB520FU/recovery.img \
    "builds/TWRP-3.7.1_16-TB520FU-portrait.img"
unset USE_PORTRAIT
```

Both variants use the same output path, so the portrait build overwrites the
landscape image unless it was copied out first. `mka installclean` only removes
the installed product files and keeps the object cache, which makes switching
between the two themes fast.

### Verifying the image

After every build, check the size and the AVB metadata before renaming or
flashing the image:

```shell
stat --format='%s bytes' out/target/product/TB520FU/recovery.img
sha256sum out/target/product/TB520FU/recovery.img
python3 external/avb/avbtool.py verify_image \
    --image out/target/product/TB520FU/recovery.img \
    --key external/avb/test/data/testkey_rsa4096.pem
```

The image size must be exactly `104857600` bytes. A successful build ends with
`build completed successfully` and produces a signed image at the path above.

## Prebuilt images

Both the landscape and the portrait recovery images are published on the
[Releases](https://github.com/lanweikayu/android_device_lenovo_TB520FU/releases)
page of this repository:

```text
TWRP-3.7.1_16-TB520FU-landscape.img   landscape (default), 104857600 bytes
TWRP-3.7.1_16-TB520FU-portrait.img    portrait, 104857600 bytes
```

Both images contain AVB metadata (signed with the TWRP test key). Compare them
against the `SHA256SUMS.txt` file attached to the release to verify the
downloads.

## Feature status

Porting status (verified on device):

- [X] ADB
- [X] Backup and restore
- [X] Data decryption (metadata/FBE)
- [X] Display
- [X] Image flashing
- [X] Partition mounting
- [X] Touch
- [X] Fastbootd
- [X] USB OTG


## Device information

- Platform: Qualcomm `pineapple` / SM8650-AB Snapdragon 8 Gen 3
- GPU: Adreno 750
- Kernel: arm64, 4 KiB page size
- Display: 12.7 inch LTPS 2944x1840 (3K), 144 Hz, 400 dpi
- Partitions: Virtual A/B with separate `recovery_a` and `recovery_b`
- Recovery partition size: 104857600 bytes
- microSD slot: none

## Installation

Before flashing, keep verified backups of both stock recovery slots. If the
recovery image cannot boot, the device may only be recoverable through Qualcomm
9008 (EDL) mode.

### Flashing through the bootloader

Use a recent Android platform-tools release on the host computer.

1. Enter the bootloader from Android or from a working recovery:

```shell
adb reboot bootloader
```

2. Confirm that the computer detects the device:

```shell
fastboot devices
```

3. Select the landscape or portrait image and flash both recovery slots:

```shell
RECOVERY_IMAGE="builds/TWRP-3.7.1_16-TB520FU-landscape.img"
fastboot flash recovery_a "$RECOVERY_IMAGE"
fastboot flash recovery_b "$RECOVERY_IMAGE"
fastboot reboot recovery
```

Replace the file name with the portrait image when needed. It is also possible
to flash only one slot, in which case the other slot keeps its current recovery.

## Credits

- [android_device_lenovo_TB321FU](https://github.com/polygraphene/android_device_lenovo_TB321FU) - the original Lenovo TB321FU device tree this project is based on.
- [TWRP-Test](https://github.com/TWRP-Test/platform_manifest_twrp_aosp)
- [TWRP](https://twrp.me/)
