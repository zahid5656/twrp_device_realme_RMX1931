# TWRP 14.1 Samurai QPR2 Alignment Report

## Active target

- Repository: `zahid5656/twrp_device_realme_RMX1931`
- Branch: `twrp-14.1`
- Recovery-task baseline: `9d2ae12d3a638e2639f24af0881f16633f2684c9`
- Device: Realme X2 Pro RMX1931/RMX1931L1 (`samurai`), SM8150/msmnile, A-only/non-A/B

## Evidence

- Current QPR2 DT: `zahid5656/android_device_realme_samurai_QPR2:infinity-x-3.12-qpr2-staging`.
- Boot/decrypt baseline: this repo `twrp-12.1`.
- TWRP 14.1 readiness reference: `twrp-14.1-a13-a16-decrypt-readiness`.
- Nayem/user reference: `zahid5656/twrp_device_realme_RMX1931_nayem:twrp-12.1L`, current HEAD `2f299bbd395bd38fa265beb21af9f7c1ff00c947`.
- `USER-PROVIDED VERIFIED RESULT`: the latest referenced Nayem 12.1L build boots on device.
- Pinned TWRP 14.1 CI source:
  - minimal manifest `cb31ddec08f495d3f70631b22140642d0045ba0d`
  - `TeamWin/android_bootable_recovery` `426b747737e7ce9e9e17da5b4d2ba883f296aec7`
  - `TeamWin/android_build` `506df226dd003a364916b6b3ee1eb3bf9064f97f`
  - `TeamWin/android_build_soong` `fed50f11f6d8dd08dd157e5450e938e2d11dbbb3`
  - `TeamWin/android_system_vold` `8e2fb2556d9d4d0b2ad5fea2e301d7c445a1ddb5`
  - `TeamWin/android_vendor_twrp` `1b4b1ff73617ff690ea525e1e1955ae9be603cbf`

## Locked QPR2 recovery contract

- `ACCEPT`: A-only dedicated recovery: `AB_OTA_UPDATER := false`, `TARGET_NO_RECOVERY := false`, `BOARD_USES_RECOVERY_AS_BOOT := false`.
- `ACCEPT`: primary `arm64 / armv8-2a-dotprod / cortex-a76`.
- `ADAPT`: secondary `arm / armv8-2a / cortex-a55`; this is required by the pinned TeamWin 14.1 ARM build rules.
- `ACCEPT`: CPUSets/SCHEDBOOST, msmnile/QCOM platform and 64-bit Binder.
- `ACCEPT`: QPR2 physical sizes: boot 100663296, recovery 83886080, cache 268435456, dtbo 25165824, system 4487905280, odm 268435456, vendor 1649410048.
- `ADAPT`: userdata default `f2fs`, with both ext4/F2FS recovery support and both `/data` fstab entries.
- `ADAPT`: `TW_USE_FSCRYPT_POLICY := 2`; TeamWin 14.1 vold selects V2 whenever the make variable is not `1`.
- `ACCEPT`: `TW_INCLUDE_RESETPROP := true` and `TARGET_KEYMASTER_WAIT_FOR_QSEE := true` for the QCOM decrypt path.
- `ACCEPT`: preserve current TWRP-14.1 `g1` ADB/MTP/sideload FunctionFS/configfs logic.

## Nayem latest-tree classification

- `DUPLICATE`: QPR2 architecture, A-only flags, partition sizes, F2FS default, fscrypt policy2, QCOM decrypt properties and recovery fstab are already represented in current target.
- `DUPLICATE`: donor `Image.gz-dtb` and `dtbo.img` are byte-identical Git blobs to current target (`f1787da066af8d694aefa35c24e6bd47f89f2ff5`, `1f3bd9b903abf7c06b1e92ee7861f9c0066cd9c1`).
- `DUPLICATE`: current target already contains the donor USB sideload/ADB fixes; keep target USB file unchanged.
- `ACCEPT`: donor commit `303328669c924dcbc7155d13797fbaaee311d39f` adds `write /sys/class/leds/vibrator/level 1` on recovery boot. Import semantically.
- `ADAPT`: donor custom theme commit `a4a76f6469369bc30e1fad5746270cb4dfd0463c`; import the latest branch's referenced `ui.xml`, `portraits.xml`, `splash.xml`, fonts and images only. Preserve font licensing. Do not re-add donor languages removed by `112f84c...`.
- `SUPERSEDED`: donor prepdecrypt debug properties/loglevel changes were removed by later `112f84c...`; do not resurrect them.
- `REJECT`: stale userdata/vendor sizes, obsolete `BOARD_BUILD_SYSTEM_ROOT_IMAGE`, duplicate Bootloader/Platform block, old AVB spelling and invalid USB `g2` creation.

## Current CI blocker

Run #22 / `31600207359` synchronized source and verified prebuilts successfully:
- kernel/DTBO SHA-256: PASS
- prebuilt structure: PASS
- patch `0001`: applied
- patch `0002`: applied
- patch `0003`: failed `git apply --check` before compilation.

The blocker is patch-stack context mismatch, not GitHub runner capacity. `0002` and `0003` both modify libtar and must be rebased/combined against the pinned TeamWin recovery source before the next build.

## Authorized active batch

1. Rebase the fscrypt-v2 libtar compatibility stack against the pinned TeamWin 14.1 source and iterate on exact build errors.
2. Import the accepted latest Nayem booted-tree changes without changing current USB logic.
3. Keep ccache/workspace but reduce CI disk pressure; do not remove `.repo` before `build.sh` applies Repo-managed source patches.
4. Build and validate `recovery.img` in CI.
5. `BUILD-VALIDATED` requires successful CI plus image structural/hash checks. `BOOT-VALIDATED` remains pending a flash/boot report for the resulting 14.1 image.

Rollback: revert recovery-task commits after `9d2ae12d3a638e2639f24af0881f16633f2684c9`.

Current validation: `SOURCE-VALIDATED`; not yet `BUILD-VALIDATED`.
