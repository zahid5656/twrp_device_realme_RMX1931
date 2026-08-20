## TWRP Device configuration for Realme X2 Pro (samurai) ##

## Features

Works:
- ADB
- Decryption of /data
- Screen brightness settings
- Correct screenshot color
- MTP
- Flashing (opengapps, roms, images and so on)
- Backup/Restore
- USB OTG

## Compilation Procedure

Create and enter a TWRP 14.1 source directory:

```bash
mkdir TWRP-14.1
cd TWRP-14.1
```

Initialize and sync the minimal TWRP 14.1 manifest, then clone this branch:

```bash
repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-14.1
repo sync
git clone --depth=1 -b twrp-14.1-a13-a16-decrypt-readiness git@github.com:zahid5656/twrp_device_realme_RMX1931.git device/realme/samurai
```

Build from the TWRP source root with the branch wrapper:

```bash
./device/realme/samurai/build.sh
```

The wrapper selects `twrp_samurai-ap2a-eng`, validates the binary structure of `Image.gz-dtb` and `dtbo.img`, applies the included idempotent TWRP 14.1 source-compatibility patches, runs `mka recoveryimage`, and validates the resulting recovery image.

Do not replace the wrapper with a direct clean-tree `source build/envsetup.sh && lunch ... && mka recoveryimage` invocation. The current TeamWin Android 14.1 recovery source still requires the compatibility patches shipped in this device tree; bypassing the wrapper can reproduce obsolete `*-ndk_platform` AIDL dependency failures.

Static prebuilt SHA-256 pin enforcement is intentionally disabled and no checksum pin file is required. This allows `Image.gz-dtb` and `dtbo.img` to be updated during development without maintaining stale hash metadata. The wrapper still requires both prebuilt files, validates the kernel gzip stream, appended FDT and DTBO header/entry structure, and after the build requires the embedded kernel and recovery DTBO to match the current tracked prebuilts byte-for-byte.

Expected Android build artifact:

```text
out/target/product/samurai/recovery.img
```

To flash the recovery image from the TWRP source root on this A-only device with a dedicated recovery partition:

```bash
fastboot flash recovery out/target/product/samurai/recovery.img
```

Extra Note: The build wrapper verifies the binary structure of the OpenELA 4.14.357 `Image.gz-dtb` and its matching two-entry `dtbo.img` before starting the Android build. It applies the included, idempotent TWRP 14.1 source-compatibility patch set. After the build it unpacks `recovery.img`, checks the partition-size limit and ramdisk integrity, and requires the embedded kernel and recovery DTBO to match the tracked prebuilts byte-for-byte. A successful build is not a substitute for booting the recovery and testing decryption on an RMX1931.
