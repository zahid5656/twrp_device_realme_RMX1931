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

Applied change: `build.sh` now adds only `asuite` to `ENVSETUP_NO_COMPLETION` before sourcing Android envsetup. Existing completion exclusions are preserved. No compiler/build warning suppression was added.

Commit: `0bfc0de1fc6e8a3c7ef004a05211dae15e7c160f`

### 3. `Trying dependencies-only mode on a non-existing device tree?` — DEFERRED AS UPSTREAM DIAGNOSTIC

TeamWin `lunch` runs RoomService in dependencies-only mode after a product is found. RoomService determines device-tree presence from `.repo/local_manifests/roomservice.xml`, not by checking whether `device/realme/samurai` physically exists. A manually cloned Samurai tree can therefore trigger this misleading message.

Decision: `DEFER`

No TeamWin RoomService source patch and no automatic mutation of the user's `.repo/local_manifests` was added solely to hide this harmless diagnostic. The physical Samurai tree and product are already resolved correctly by the build system.

### 4. Current prebuilt hashes are stale — BLOCKER FOR WRAPPER VALIDATION

`prebuilt/SHA256SUMS` still contains hashes for an older prebuilt set, while the branch currently contains updated `Image.gz-dtb` and `dtbo.img` binaries.

Decision: `BLOCKER`

No fabricated SHA-256 values were written. `prebuilt/SHA256SUMS` must be updated from the exact current binary payloads before the wrapper's prebuilt validation can pass.

Current tracked binary sizes at audit time:

- `prebuilt/Image.gz-dtb`: 19,166,656 bytes
- `prebuilt/dtbo.img`: 496,493 bytes

## Applied remote changes

1. `REPORT.md`
   - recorded the source evidence, exact scope, rollback path, and unresolved hash blocker.

2. `README.md`
   - removed the direct `envsetup/lunch/mka` path as the supported build procedure;
   - made `./device/realme/samurai/build.sh` the supported entry point;
   - documented why the wrapper is required;
   - removed the hard-coded `/home/titan/TWRP-14.1` path;
   - corrected the recovery artifact path used by the flash example.

3. `build.sh`
   - preserved the eight existing compatibility patches and prebuilt/recovery validation logic;
   - suppressed only the absent Asuite completion probe through `ENVSETUP_NO_COMPLETION`;
   - preserved all compiler and build warnings.

4. `prebuilt/SHA256SUMS`
   - intentionally unchanged until exact current SHA-256 values are available.

## Rollback

Revert, in reverse order if required:

- `0bfc0de1fc6e8a3c7ef004a05211dae15e7c160f` — build wrapper environment fix
- `dcb19f71489ee5ead8788eabdb7020f39f201e86` — README build-path fix
- `3a6d06f09845e4c9a34ec77eae02ba2695692380` — initial report

## Validation required

After updating `prebuilt/SHA256SUMS`, run from the TWRP source root:

```bash
./device/realme/samurai/build.sh
```

Expected source-level behavior:

- current prebuilt hashes validate;
- the eight compatibility patches are applied or detected as already applied;
- the Asuite completion warning is not emitted by the wrapper;
- build target resolves to `twrp_samurai-ap2a-eng`;
- obsolete `*-ndk_platform` AIDL dependency errors do not reappear;
- resulting `recovery.img` is structurally validated by the wrapper.

No new `BUILD-VALIDATED`, `BOOT-VALIDATED`, `FEATURE-VALIDATED`, or `RELEASE-VALIDATED` claim is made by this remote edit pass.
