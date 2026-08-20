# TWRP 14.1 Samurai Remote Fix Report

## Scope

Branch: `twrp-14.1-a13-a16-decrypt-readiness`

Audit start HEAD: `f2929184f23b6b906ba54e56091e4dc70f2ad765`

Task: fix the currently observed TWRP 14.1 build-path problems without changing the device crypto policy, kernel payload contract, partition layout, or unrelated recovery behavior.

## Result classification

`SOURCE-VALIDATED`

The fixes below are source-validated only. They are not a new build, boot, or decryption validation result.

## Verified problems

### 1. README bypasses the branch compatibility patch path

The branch contains eight idempotent TWRP 14.1 compatibility patches and `build.sh` applies them before invoking the Android build. The README instead instructed users to call `source build/envsetup.sh`, `lunch`, and `mka recoveryimage` directly.

That direct path bypasses `apply_source_compat_patches()` and reproduces the observed fatal dependency failure for obsolete Android 14 AIDL NDK module names, including `android.security.apc-ndk_platform`.

Decision: `ACCEPT` — make `build.sh` the documented build entry point.

### 2. Minimal manifest intentionally removes Asuite, while Android envsetup still probes its completion script

The TWRP minimal manifest removes `tools/asuite`. Android 14 `build/envsetup.sh` still probes `tools/asuite/asuite.sh` and emits a warning when it is absent.

Decision: `ACCEPT` — set `ENVSETUP_NO_COMPLETION` to include `asuite` inside the build wrapper before sourcing `build/envsetup.sh`. This suppresses only the unavailable Asuite shell-completion probe; it does not suppress compiler/build warnings.

### 3. `Trying dependencies-only mode on a non-existing device tree?`

TeamWin `lunch` runs RoomService in dependencies-only mode after a product is found. RoomService determines device-tree presence from `.repo/local_manifests/roomservice.xml`, not by checking whether `device/realme/samurai` physically exists. A manually cloned Samurai tree can therefore trigger this misleading message.

Decision: `DEFER` — do not patch TeamWin RoomService or mutate the user's `.repo/local_manifests` from the device build wrapper solely to hide a harmless diagnostic. The documented wrapper remains the supported build path.

### 4. Current prebuilt hashes are stale

`prebuilt/SHA256SUMS` still contains hashes for an older prebuilt set, while the branch currently contains updated `Image.gz-dtb` and `dtbo.img` binaries.

Decision: `BLOCKER` — do not fabricate SHA-256 values. Update `prebuilt/SHA256SUMS` only after hashing the exact current binary payloads.

## Approved remote changes

1. `README.md`
   - replace the direct `envsetup/lunch/mka` build path with `./device/realme/samurai/build.sh`;
   - explain that the wrapper applies the required TWRP 14.1 compatibility patches before compilation;
   - retain the correct `twrp_samurai-ap2a-eng` target information.

2. `build.sh`
   - preserve all existing prebuilt validation and compatibility-patch logic;
   - add narrowly scoped Asuite completion suppression through `ENVSETUP_NO_COMPLETION` before sourcing Android envsetup;
   - do not add broad warning suppression.

3. `prebuilt/SHA256SUMS`
   - no write in this fix commit until the exact current binary SHA-256 values are available.

## Rollback

Revert the follow-up fix commit that modifies `README.md` and `build.sh`. `REPORT.md` is documentation-only and may be reverted independently if required.

## Validation required after fix

Run from the TWRP source root:

```bash
./device/realme/samurai/build.sh
```

Expected source-level behavior:

- the eight compatibility patches are applied or detected as already applied;
- the Asuite completion warning is not emitted by the wrapper;
- build target resolves to `twrp_samurai-ap2a-eng`;
- stale `prebuilt/SHA256SUMS` may intentionally stop the wrapper until its hashes are updated.

No `BUILD-VALIDATED`, `BOOT-VALIDATED`, `FEATURE-VALIDATED`, or `RELEASE-VALIDATED` claim is made by this report.
