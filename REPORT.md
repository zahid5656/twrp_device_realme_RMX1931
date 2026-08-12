# TWRP 14.1 Samurai QPR2 Alignment Report

## Active target

- Repository: `zahid5656/twrp_device_realme_RMX1931`
- Branch: `twrp-14.1`
- Pre-alignment HEAD for this batch: `9d2ae12d3a638e2639f24af0881f16633f2684c9`
- Device: Realme X2 Pro RMX1931/RMX1931L1 (`samurai`), SM8150/msmnile, A-only/non-A/B

## Evidence used

- Current QPR2 DT: `zahid5656/android_device_realme_samurai_QPR2:infinity-x-3.12-qpr2-staging`, `BoardConfig.mk` blob `85c155601476ebacb76242b290c21e70ff493851`.
- Boot/decrypt baseline: this repo `twrp-12.1`.
- Android 14 readiness reference: this repo `twrp-14.1-a13-a16-decrypt-readiness`.
- Nayem recovery reference: `zahid5656/twrp_device_realme_RMX1931_nayem:twrp-12.1L`.
- Pinned latest TWRP 14.1 CI source from run 12:
  - minimal manifest `cb31ddec08f495d3f70631b22140642d0045ba0d`
  - `TeamWin/android_bootable_recovery` `426b747737e7ce9e9e17da5b4d2ba883f296aec7`
  - `TeamWin/android_build` `506df226dd003a364916b6b3ee1eb3bf9064f97f`
  - `TeamWin/android_build_soong` `fed50f11f6d8dd08dd157e5450e938e2d11dbbb3`
  - `TeamWin/android_system_vold` `8e2fb2556d9d4d0b2ad5fea2e301d7c445a1ddb5`
  - `TeamWin/android_vendor_twrp` `1b4b1ff73617ff690ea525e1e1955ae9be603cbf`

## Current findings / classifications

- `ACCEPT`: primary arch `arm64 / armv8-2a-dotprod / cortex-a76`.
- `ADAPT`: secondary arch must be `arm / armv8-2a / cortex-a55`. TWRP 14.1 CI proved `armv8-2a + cortex-a76` invalid; pinned TeamWin build explicitly accepts `cortex-a55` as ARMv8.2-A.
- `ACCEPT`: `TARGET_USES_64_BIT_BINDER := true`, present in the proven 12.1/Nayem baseline and consistent with SM8150.
- `ACCEPT`: explicit A-only dedicated-recovery flags `TARGET_NO_RECOVERY := false` and `BOARD_USES_RECOVERY_AS_BOOT := false`.
- `ACCEPT`: QPR2 physical partition sizes already synchronized from current DT: boot 100663296, recovery 83886080, cache 268435456, dtbo 25165824, system 4487905280, odm 268435456, vendor 1649410048.
- `ACCEPT`: keep `TW_USE_FSCRYPT_POLICY := 1` because the current TWRP-14.1 compatibility patch stack is built and tested around policy-v1 recovery structs; Android `/data` remains FBE v2 metadata encryption independently.
- `DEFER`: Nayem `TW_USE_FSCRYPT_POLICY := 2` until the TWRP-14.1 patch stack is converted and build/runtime validated.
- `DEFER`: `TARGET_KEYMASTER_WAIT_FOR_QSEE := true` until a current TWRP-14.1 consumer is proven.
- `REJECT`: Nayem USB `g2` configfs directories because the gadget itself is never created.
- `ACCEPT`: preserve current `twrp-14.1` `g1` ADB/MTP/sideload FunctionFS/configfs logic unchanged.
- `DEFER`: supplied retrofit-dynamic-partition `BOARD_SUPER_*` values. Current QPR2 Samurai DT has no `BOARD_SUPER_*` contract and mounts physical system/vendor partitions. Do not invent a virtual super layout in recovery until exact Infinity-X QPR2 artifact metadata/partition map proves it.

## Current CI evidence

- Run 11 reached current TWRP 14.1 source and failed because `.repo` metadata was removed before applying source patches; fixed.
- Run 12 preserved Repo metadata and verified pinned kernel/DTBO hashes, then failed at BoardConfig validation:
  `Incorrect TARGET_2ND_ARCH_VARIANT, armv8-2a. Use armv8-a instead.`
  This occurs because the current tree paired ARMv8.2-A with `cortex-a76`; pinned TeamWin Android 14 build recognizes `cortex-a55` as an ARMv8.2-A 32-bit core.

## Authorized next batch

1. Correct secondary CPU variant to `cortex-a55` while keeping `armv8-2a`.
2. Add explicit A-only dedicated-recovery flags and 64-bit Binder flag.
3. Add RMX1931L1 to recovery OTA assert.
4. Preserve current TWRP-14.1 USB/ADB/sideload implementation.
5. Push and run CI using the pinned/latest TWRP 14.1 manifest branch.
6. Iterate only on exact build errors.

Rollback: revert commits after `9d2ae12d3a638e2639f24af0881f16633f2684c9`.

Validation before this batch: `SOURCE-VALIDATED`; not yet `BUILD-VALIDATED`.
