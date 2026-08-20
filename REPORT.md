# TWRP 14.1 Samurai Remote Fix Report

## Scope

Branch: `twrp-14.1-a13-a16-decrypt-readiness`

Audit start HEAD: `f2929184f23b6b906ba54e56091e4dc70f2ad765`

Task: fix the currently observed TWRP 14.1 build-path problems without changing the device crypto policy, kernel payload contract, partition layout, or unrelated recovery behavior.

## Result classification

`SOURCE-VALIDATED`

The fixes below are source-validated only. They are not a new build, boot, or decryption validation result.

## Verified problems and disposition

### 1. README bypassed the branch compatibility patch path — FIXED

The branch contains eight idempotent TWRP 14.1 compatibility patches and `build.sh` applies them before invoking the Android build. The old README instructed users to call `source build/envsetup.sh`, `lunch`, and `mka recoveryimage` directly.

That direct path bypasses `apply_source_compat_patches()` and reproduces the observed fatal dependency failure for obsolete Android 14 AIDL NDK module names, including `android.security.apc-ndk_platform`.

Decision: `ACCEPT`

Applied change: `README.md` now makes `./device/realme/samurai/build.sh` the documented build entry point and explicitly warns that direct clean-tree `mka recoveryimage` bypasses the required compatibility patch set.

Commit: `dcb19f71489ee5ead8788eabdb7020f39f201e86`

### 2. Minimal manifest intentionally removes Asuite, while Android envsetup still probes its completion script — FIXED IN WRAPPER

The TWRP minimal manifest removes `tools/asuite`. Android 14 `build/envsetup.sh` still probes `tools/asuite/asuite.sh` and emits a warning when it is absent.

Decision: `ACCEPT`

Applied change: `build.sh` adds only `asuite` to `ENVSETUP_NO_COMPLETION` before sourcing Android envsetup. Existing completion exclusions are preserved. No compiler/build warning suppression was added.

Commit: `0bfc0de1fc6e8a3c7ef004a05211dae15e7c160f`

### 3. `Trying dependencies-only mode on a non-existing device tree?` — DEFERRED AS UPSTREAM DIAGNOSTIC

TeamWin `lunch` runs RoomService in dependencies-only mode after a product is found. RoomService determines device-tree presence from `.repo/local_manifests/roomservice.xml`, not by checking whether `device/realme/samurai` physically exists. A manually cloned Samurai tree can therefore trigger this misleading message.

Decision: `DEFER`

No TeamWin RoomService source patch and no automatic mutation of the user's `.repo/local_manifests` is added solely to hide this harmless diagnostic. The physical Samurai tree and product are already resolved correctly by the build system.

### 4. Prebuilt SHA-256 pin verification — APPROVED FOR PERMANENT DISABLE

The tracked `prebuilt/SHA256SUMS` values no longer match the updated `Image.gz-dtb` and `dtbo.img`. On GCP, the user disabled only the `sha256sum -c SHA256SUMS` gate locally and verified that the wrapper then continued successfully through prebuilt structural validation and into the Android build.

`USER-PROVIDED VERIFIED RESULT`: after the local checksum-gate bypass, the wrapper reported:

- `Prebuilt structure: PASS`
- kernel size 19,166,656 bytes
- appended DTB size 451,191 bytes
- DTBO entry count 2
- all eight TWRP 14.1 compatibility patches applied
- product configuration reached `twrp_samurai-ap2a-eng`
- Android build started.

Decision: `ACCEPT`

Approved remote change:

- remove `SHA256SUMS` from the required prebuilt-file gate;
- remove the enforced `sha256sum -c SHA256SUMS` step;
- print `SHA-256 prebuilt pin verification: DISABLED`;
- retain gzip integrity, appended FDT magic/size, DTBO header/magic/entry/page validation;
- retain post-build byte-for-byte comparison of the embedded kernel and recovery DTBO against the exact tracked prebuilts;
- retain final image/ramdisk/partition-size validation.

This disables only stale hash pin enforcement. It does not disable binary structural validation or final payload identity validation.

## Remote change plan

1. `build.sh`
   - change the required prebuilt list from `Image.gz-dtb dtbo.img SHA256SUMS` to `Image.gz-dtb dtbo.img`;
   - replace `sha256sum -c SHA256SUMS` with the explicit disabled-status message;
   - preserve every structural and post-build validation check.

2. `README.md`
   - remove the instruction to regenerate `prebuilt/SHA256SUMS` before each updated-prebuilt build;
   - document that checksum pin enforcement is disabled while structural and embedded-payload checks remain mandatory.

3. `prebuilt/SHA256SUMS`
   - leave untouched; it is no longer part of the enforced build contract.

## Rollback

Revert the checksum-policy follow-up commits to restore SHA-256 pin enforcement. Earlier wrapper fixes may be reverted independently if required.

## Validation after remote change

Run from the TWRP source root:

```bash
./device/realme/samurai/build.sh
```

Expected behavior:

- wrapper does not execute `sha256sum -c SHA256SUMS`;
- wrapper prints `SHA-256 prebuilt pin verification: DISABLED`;
- kernel/DTBO structural validation must still pass;
- the eight compatibility patches are applied or detected as already applied;
- build target resolves to `twrp_samurai-ap2a-eng`;
- resulting `recovery.img` must still pass structure, ramdisk, size and embedded kernel/DTBO identity checks.

No new `BUILD-VALIDATED`, `BOOT-VALIDATED`, `FEATURE-VALIDATED`, or `RELEASE-VALIDATED` claim is made by this remote edit pass.
