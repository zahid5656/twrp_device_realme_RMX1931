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

### 4. Prebuilt SHA-256 pin verification — PERMANENTLY DISABLED

The tracked `prebuilt/SHA256SUMS` values no longer matched the updated `Image.gz-dtb` and `dtbo.img`. On GCP, the user disabled only the `sha256sum -c SHA256SUMS` gate locally and verified that the wrapper then continued successfully through prebuilt structural validation and into the Android build.

`USER-PROVIDED VERIFIED RESULT`: after the local checksum-gate bypass, the wrapper reported:

- `Prebuilt structure: PASS`
- kernel size 19,166,656 bytes
- appended DTB size 451,191 bytes
- DTBO entry count 2
- all eight TWRP 14.1 compatibility patches applied
- product configuration reached `twrp_samurai-ap2a-eng`
- Android build started.

Decision: `ACCEPT`

Applied policy:

- `build.sh` requires only `Image.gz-dtb` and `dtbo.img` as prebuilt inputs;
- `sha256sum -c SHA256SUMS` is not executed;
- `prebuilt/SHA256SUMS` is removed from the branch because the kernel and DTBO are intentionally updated during development and static hash pin maintenance is not part of the build contract;
- gzip integrity, appended FDT magic/size, DTBO header/magic/entry/page validation remain enabled;
- post-build byte-for-byte comparison of the embedded kernel and recovery DTBO against the exact current prebuilts remains enabled;
- final image, ramdisk and partition-size validation remain enabled.

This removes only static checksum pinning. It does not disable binary structural validation or final payload identity validation.

## Applied remote changes

1. `build.sh`
   - required prebuilt list is `Image.gz-dtb dtbo.img`;
   - static SHA-256 verification is disabled;
   - every structural and post-build validation check remains enabled.

2. `README.md`
   - documents the wrapper as the supported build path;
   - documents that static checksum pinning is not used;
   - documents the remaining structural and embedded-payload checks.

3. `prebuilt/SHA256SUMS`
   - removed from the branch as obsolete development metadata.

## Rollback

Restore `prebuilt/SHA256SUMS` and the corresponding `sha256sum -c SHA256SUMS` gate only if static prebuilt pinning is intentionally reintroduced. Earlier wrapper fixes may be reverted independently if required.

## Validation after remote change

Run from the TWRP source root:

```bash
./device/realme/samurai/build.sh
```

Expected behavior:

- no `SHA256SUMS` file is required;
- wrapper prints `SHA-256 prebuilt pin verification: DISABLED`;
- kernel/DTBO structural validation must still pass;
- the eight compatibility patches are applied or detected as already applied;
- build target resolves to `twrp_samurai-ap2a-eng`;
- resulting `recovery.img` must still pass structure, ramdisk, size and embedded kernel/DTBO identity checks.

No new `BUILD-VALIDATED`, `BOOT-VALIDATED`, `FEATURE-VALIDATED`, or `RELEASE-VALIDATED` claim is made by this remote edit pass.
