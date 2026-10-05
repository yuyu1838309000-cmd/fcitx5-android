# Neryven IME — Upstream Compatibility Notes

This file records the smallest intentional deviations from upstream that matter when rebasing or syncing `fcitx5-android/fcitx5-android`.

## V0 baseline

- Base commit when V0 started: `e6199a28`.
- Upstream remote: `fcitx5-android/fcitx5-android`.
- Source namespace / Kotlin packages / JNI class names remain upstream `org.fcitx.fcitx5.android`.
- Release applicationId: `com.neryven.ime`.
- Debug applicationId: `com.neryven.ime.debug`.
- V0 does not rewrite the plugin package prefix `org.fcitx.fcitx5.android.plugin.*`.

## Intentional code differences

### Main app identity
- `gradle.properties` defines `mainApplicationId=com.neryven.ime`.
- `app/build.gradle.kts` reads `mainApplicationId` instead of hardcoding the upstream applicationId.
- Debug keeps the upstream `.debug` suffix.
- Debug display label is temporarily `Neryven IME (Debug)` in every locale; this is intentional for V0 development builds.
- Release still uses the upstream Fcitx5 / 小企鹅输入法 label resources until the final product name is chosen; V0 does not ship a release build.

### Plugin ↔ main-app identity
- `AndroidPluginAppConventionPlugin.kt` derives `MAIN_APPLICATION_ID` and the `mainApplicationId` manifest placeholder from the project property for release/debug variants.
- `lib/plugin-base/src/main/AndroidManifest.xml` uses `${mainApplicationId}` for package visibility, metadata keys and plugin manifest actions.
- The debug plugin-base manifest no longer hardcodes the upstream release/debug package IDs; it only merges debug-specific activity attributes.
- Plugin package names themselves are intentionally not renamed in V0.

## Upstream-sync hotspots

After pulling/rebasing upstream, inspect these files before accepting the sync:

1. `app/build.gradle.kts`
2. `gradle.properties`
3. `build-logic/convention/src/main/kotlin/AndroidPluginAppConventionPlugin.kt`
4. `lib/plugin-base/src/main/AndroidManifest.xml`
5. `lib/plugin-base/src/debug/AndroidManifest.xml`

The main app currently gets its debug identity from the existing AGP `.debug` suffix, while `AndroidPluginAppConventionPlugin.kt` appends `.debug` explicitly for repo plugins. If upstream changes either debug-ID rule, update and verify both sides together.

Also search for new hardcoded references to:
- `org.fcitx.fcitx5.android` when used as an **application identity** rather than source namespace;
- plugin IPC permission/action/metadata names;
- provider authorities derived from applicationId;
- new JNI assumptions tied to applicationId.

Do not mass-replace `org.fcitx.fcitx5.android`: most occurrences are legitimate source namespace / class names and are intentionally preserved.

## Verified V0 evidence

- Unmodified-package arm64 debug baseline build: PASS.
- Renamed `com.neryven.ime.debug` arm64 debug build: PASS.
- `:plugin:clipboard-filter:assembleDebug`: PASS.
- Built APK package: `com.neryven.ime.debug`.
- APK SHA256: `0B06FA3C6B1A78C63174373072B482A60BFDF08372A3512F388DDCF6680450C5`.
- Real-device install/enable/default-IME flow: PASS.
- Real-device basic Pinyin/English input path: PASS.
- `git diff --check`: PASS.

## Submodule note

The V0 Pinyin/English build path has all required submodules initialized. Two nested optional paths were still uninitialized in the last recursive status check:
- `lib/fcitx5/src/main/cpp/fcitx5/third_party/yoga`
- `plugin/anthy/src/main/cpp/anthy-cmake/anthy-unicode`

Neither is part of the verified V0 Pinyin/English path. `yoga` is only required by the desktop X11/Wayland path, which the Android build disables; `anthy-unicode` is only required when validating the Anthy plugin. If upstream changes begin to require either one, or when validating the full plugin/desktop matrix, initialize and verify them before claiming a complete recursive checkout.

## Local build tooling

The local `.tooling/` directory is ignored and contains local Gradle/download cache material only. It is not product source and must not be committed.

The project intentionally keeps upstream wrapper configuration unchanged. Local mirror/cache workarounds used for slow downloads are environment-specific and should not become source changes unless reproducibility requires it.
