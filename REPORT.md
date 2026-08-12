# TWRP 14.1 Samurai QPR2 Alignment Report

## Active target

- Repository: `zahid5656/twrp_device_realme_RMX1931`
- Branch: `twrp-14.1`
- Pre-alignment HEAD for this recovery task: `9d2ae12d3a638e2639f24af0881f16633f2684c9`
- Device: Realme X2 Pro RMX1931/RMX1931L1 (`samurai`), SM8150/msmnile, A-only/non-A/B

## Evidence used

- Current QPR2 DT: `zahid5656/android_device_realme_samurai_QPR2:infinity-x-3.12-qpr2-staging`.
- Boot/decrypt baseline: this repo `twrp-12.1`.
- Android 14 readiness reference: this repo `twrp-14.1-a13-a16-decrypt-readiness`.
- Nayem recovery reference: `zahid5656/twrp_device_realme_RMX1931_nayem:twrp-12.1L`, BoardConfig blob `fae64212521c84cb9af4572abbddd10cc45ed5b8`.
- Pinned TWRP 14.1 CI source from run 12:
  - minimal manifest `cb31ddec08f495d3f70631b22140642d0045ba0d`
  - `TeamWin/android_bootable_recovery` `426b747737e7ce9e9e17da5b4d2ba883f296aec7`
  - `TeamWin/android_build` `506df226dd003a364916b6b3ee1eb3bf9064f97f`
  - `TeamWin/android_build_soong` `fed50f11f6d8dd08dd157e5450e938e2d11dbbb3`
  - `TeamWin/android_system_vold` `8e2fb2556d9d4d0b2ad5fea2e301d7c445a1ddb5`
  - `TeamWin/android_vendor_twrp` `1b4b1ff73617ff690ea525e1e1955ae9be603cbf`

## Current findings / classifications

- `ACCEPT`: primary arch `arm64 / armv8-2a-dotprod / cortex-a76`.
- `ADAPT`: secondary arch `arm / armv8-2a / cortex-a55`; pinned TeamWin Android 14 build accepts `cortex-a55` as ARMv8.2-A.
- `ACCEPT`: `TARGET_USES_64_BIT_BINDER := true`.
- `ACCEPT`: explicit A-only dedicated-recovery flags `TARGET_NO_RECOVERY := false` and `BOARD_USES_RECOVERY_AS_BOOT := false`.
- `ACCEPT`: QPR2 physical partition sizes: boot 100663296, recovery 83886080, cache 268435456, dtbo 25165824, system 4487905280, odm 268435456, vendor 1649410048.
- `ADAPT`: userdata default remains `f2fs`; keep both ext4/F2FS image support and both `/data` fstab entries.
- `ADAPT`: QPR2 recovery crypto target is `TW_USE_FSCRYPT_POLICY := 2`; pinned TeamWin vold selects policy-v2 whenever the make variable is not `1`.
- `ACCEPT`: add explicit `TW_INCLUDE_RESETPROP := true`. TeamWin QCOM TWRP common enables resetprop for QCOM decryption, and Samurai packages `qcom_decrypt`/`qcom_decrypt_fbe`.
- `ADAPT`: add `TARGET_KEYMASTER_WAIT_FOR_QSEE := true` from Nayem `twrp-12.1L` as the msmnile/QCOM decrypt compatibility setting; source/build acceptance will be checked by CI and functional effect requires on-device decrypt validation.
- `DUPLICATE`: Nayem `device.mk` is functionally already represented in current `twrp-14.1` for shipping API 28, asserts, qcom decrypt packages, display recovery libs and AIDL haptics.
- `REJECT`: Nayem runtime CPU overrides `TARGET_CPU_VARIANT_RUNTIME := kryo485` and `TARGET_2ND_CPU_VARIANT_RUNTIME := cortex-a76`; current QPR2/TeamWin-validated compile variants are explicit and the secondary override conflicts with the selected Cortex-A55 32-bit target.
- `REJECT`: `BOARD_BUILD_SYSTEM_ROOT_IMAGE := true`; current TWRP 14.1 build rejects this obsolete setting.
- `REJECT`: Nayem stale `BOARD_USERDATAIMAGE_PARTITION_SIZE := 12884901888`, stale vendor size `1711276032`, and donor AVB `--flag 3`; current QPR2 physical layout and AVB contract remain authoritative.
- `REJECT`: Nayem USB `g2` configfs directories because the gadget itself is never created.
- `ACCEPT`: preserve current `twrp-14.1` `g1` ADB/MTP/sideload FunctionFS/configfs logic unchanged.
- `DEFER`: supplied retrofit-dynamic-partition `BOARD_SUPER_*` values until exact Infinity-X QPR2 artifact/partition metadata proves recovery must model them.

## Current CI evidence

- Run 12 synchronized and pinned current TWRP 14.1 source, verified kernel/DTBO hashes, then failed at BoardConfig validation because `armv8-2a` was paired with `cortex-a76` for the 32-bit architecture.
- The CPU pairing was corrected to `armv8-2a / cortex-a55`.
- F2FS default and policy-v2 libtar compatibility are now in the active source and require the next successful CI build for `BUILD-VALIDATED` status.

## Authorized active batch

1. Keep secondary CPU variant `cortex-a55` with `armv8-2a`.
2. Keep `BOARD_USERDATAIMAGE_FILE_SYSTEM_TYPE := f2fs`; retain ext4 + F2FS runtime support/fstab entries.
3. Keep `TW_USE_FSCRYPT_POLICY := 2` and the v2 libtar compatibility layer.
4. Add the validated Nayem/QCOM recovery properties `TW_INCLUDE_RESETPROP := true` and `TARGET_KEYMASTER_WAIT_FOR_QSEE := true`.
5. Preserve current TWRP-14.1 USB/ADB/MTP/sideload implementation.
6. Push and run CI against the pinned/latest TWRP 14.1 manifest branch.
7. Iterate only on exact build errors; do not claim decrypt until runtime tested on device.

Rollback: revert recovery-task commits after `9d2ae12d3a638e2639f24af0881f16633f2684c9`.

Validation before this property batch: `SOURCE-VALIDATED`; not yet `BUILD-VALIDATED`.
