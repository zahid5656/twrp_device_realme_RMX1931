# TWRP device configuration for Realme X2 Pro (samurai)

Target device: RMX1931 / RMX1931CN, Qualcomm SM8150 (msmnile), A-only.

## Current target

This branch tracks the latest available minimal TWRP AOSP manifest branch:
`twrp-14.1`. The manifest is Android 14 based and replaces selected AOSP
projects with TeamWin `android-14.1` branches. CI records the exact manifest and
project revisions used for every build.

The recovery tree keeps the Android 13-16 metadata-FBE path derived from the
known working TWRP 12.1 Samurai recovery and the build-validated TWRP 14.1
readiness branch. Android 13/14/15/16 decryption on this TWRP 14.1 branch still
requires physical-device validation before it is considered feature validated.

## Samurai QPR2 alignment

- A-only / non-A/B recovery layout (`AB_OTA_UPDATER := false`)
- Boot image header v1
- 4096-byte kernel page size
- `Image.gz-dtb` prebuilt kernel
- Separate recovery DTBO
- Dedicated recovery partition
- SM8150 architecture: `armv8-2a-dotprod` / `cortex-a76`
- 32-bit secondary architecture: `armv8-2a` / `cortex-a76`
- CPUsets and schedboost enabled for the recovery build
- ext4 and f2fs userdata recovery support
- metadata-FBE/QCOM decryption stack with `TW_USE_FSCRYPT_POLICY := 1`

Physical partition sizes are aligned with the current Samurai Android 16 QPR2
device tree instead of the older hard-coded recovery values.

## Global F.14 firmware prerequisite

For this project, `Global_F14_firmware_only.zip` is a mandatory prerequisite
when installing a custom ROM on the Realme X2 Pro. Flash it as required by the
ROM installation procedure before/alongside the custom ROM.

The verified updater writes the following A-only `/dev/block/bootdevice/by-name`
partitions:

`abl`, `xbl`, `xbl_config`, `splash`, `oppo_sec`, `modem`, `oppostanvbk`,
`dsp`, `hyp`, `storsec`, `devcfg`, `tz`, `apdp`, `cmnlib`, `aop`, `msadp`,
`qupfw`, `bluetooth`, `cmnlib64`, and `keymaster`.

This recovery tree must therefore preserve the Samurai bootdevice/by-name path
and must not import A/B slot-suffix firmware flashing logic.

## Build

```sh
repo init --depth=1 -u https://github.com/minimal-manifest-twrp/platform_manifest_twrp_aosp.git -b twrp-14.1
repo sync
git clone https://github.com/zahid5656/twrp_device_realme_RMX1931.git -b twrp-14.1 device/realme/samurai
bash device/realme/samurai/build.sh
```

The build wrapper uses `twrp_samurai-ap2a-eng`, applies the included TWRP 14.1
source-compatibility patches idempotently, validates the prebuilt kernel/DTBO
structure, builds `recovery.img`, unpacks the result, checks the recovery
partition-size limit and ramdisk integrity, and requires the embedded kernel and
recovery DTBO to match the device-tree prebuilts byte-for-byte.

GitHub Actions uses the same `build.sh` path as a local build. It uploads the
recovery image, build log, exact pinned manifest and SHA-256 metadata as workflow
artifacts. It does not create a release or tag automatically.

## Required on-device validation

Before release, validate recovery boot, ADB, MTP, sideload, USB-OTG,
`/metadata`, F.14 firmware ZIP installation, encrypted `/data` decryption on
Android 13/14/15/16, ext4/f2fs userdata where applicable, image flashing,
backup/restore, format-data, and normal/recovery reboot paths.
