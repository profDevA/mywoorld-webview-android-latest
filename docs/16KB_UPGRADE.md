# 16 KB Page Size Fix — React Native Upgrade

## Why

Google Play requires apps targeting Android 15+ to support **16 KB memory page
sizes** on 64-bit devices. The failing pieces are the prebuilt native `.so`
libraries shipped inside React Native (Hermes, the RN core, JSC) and native
modules — their ELF `LOAD` segments must be aligned to 16 KB (`2**14`).

On **React Native 0.73**, the RN-provided native libraries are only 4 KB-aligned
and **cannot be re-aligned from app Gradle config**. Per the React Native team,
the only fix is to upgrade to **RN 0.77+**, where all internals are recompiled
16 KB-aligned. This project was upgraded to **RN 0.85.3**.

## What changed

### JS / dependencies (`package.json`)
| Package | From | To |
| --- | --- | --- |
| react-native | 0.73.9 | 0.85.3 |
| react | 18.2.0 | 19.2.3 |
| @react-navigation/native | ^6.1.17 | ^7.3.0 |
| @react-navigation/stack | ^6.4.0 | ^7.10.1 |
| react-native-screens | (none) | ^4.25.2 (new — required by stack v7) |
| react-native-gesture-handler | ^2.17.1 | ^3.0.0 |
| react-native-safe-area-context | ^4.10.8 | ^5.8.0 |
| react-native-webview | ^13.10.5 | ^13.16.1 |
| react-native-permissions | ^5.4.2 | ^5.5.2 |

Dev tooling (Babel, CLI, `@react-native/*` presets, TypeScript, types) was
bumped to the 0.85.3 template versions.

### Android (this is what satisfies the Play requirement)
- `build.gradle`: `compileSdk`/`targetSdk` 35→36, `buildTools` 36.0.0,
  **NDK 27.1.12297006**, Kotlin 2.1.20.
- `gradle.properties`: `newArchEnabled=true`, removed `enableJetifier`,
  added `edgeToEdgeEnabled=false`.
- `gradle-wrapper.properties`: Gradle **9.3.1**.
- `app/build.gradle`: new `autolinkLibrariesWithApp()`, dropped Flipper and the
  legacy `native_modules.gradle` apply, updated JSC flavor.
- `settings.gradle`: plugin-based autolinking.
- `MainApplication.kt`: simplified to `loadReactNative(this)`.
- Removed `app/src/debug/AndroidManifest.xml` (cleartext is now a build-injected
  manifest placeholder); added `supportsRtl` + `usesCleartextTraffic` placeholder
  to the main manifest.

### iOS
See "iOS status" below — not yet migrated.

## Build & verify (Android)

> Requires NDK `27.1.12297006` installed via Android Studio SDK Manager.
> The Gradle wrapper auto-downloads Gradle 9.3.1 on first build.

```bash
# from project root
yarn install

# build a release App Bundle / APK
cd android
./gradlew clean
./gradlew :app:bundleRelease     # AAB for Play
./gradlew :app:assembleRelease   # APK for local verification
```

### Confirm 16 KB alignment

**For an APK** (from the Android SDK build-tools 35.0.0+ folder):

```powershell
zipalign.exe -c -P 16 -v 4 app-release.apk   # expect "Verification successful"
```

**For an AAB** (or to inspect the `.so` files directly), check that every ELF
`LOAD` segment is aligned to `2**14` (16 KB) using the NDK's `llvm-objdump`:

```powershell
$ndk = "$env:ANDROID_HOME\ndk\27.1.12297006\toolchains\llvm\prebuilt\windows-x86_64\bin\llvm-objdump.exe"
$lib = "android/app/build/intermediates/merged_native_libs/release/mergeReleaseNativeLibs/out/lib/arm64-v8a"
Get-ChildItem $lib -Filter *.so | ForEach-Object {
  $a = (& $ndk -p $_.FullName | Select-String "LOAD" | ForEach-Object { ($_ -split 'align ')[1] } | Select-Object -Unique)
  "{0,-45} {1}" -f $_.Name, $a
}
```

Every library should report `2**14`. You can also use
**Android Studio → Build → Analyze APK…** and confirm the *Alignment* column
shows no warnings on the `arm64-v8a` / `x86_64` `.so` files.

### ✅ Verified

On `:app:bundleRelease` (RN 0.85.3 + NDK 27), all 15 bundled `arm64-v8a`
libraries — including `libreactnative.so`, `libhermesvm.so`, `libjsi.so`, and
the codegen/native-module libs — reported `2**14` alignment with **0 failures**.
The build is 16 KB-compliant. (`x86_64` comes from the same NDK 27 build and is
identical; `arm64-v8a` + `x86_64` are the only ABIs the policy covers.)

## Release & upload

The signed release App Bundle is produced via **Android Studio → Build →
Generate Signed Bundle / APK → Android App Bundle**, signed with the current
**upload key** (keystore `mywld`, alias `mywld`, located alongside the repo).
See [`SETUP.md`](./SETUP.md#release-build-signed-aab) for the full flow.

- `versionCode` was bumped **3 → 4** (`versionName` `2.1` → `2.2`) in
  `android/app/build.gradle`, since versionCode 3 is already live on Play.
- After uploading the new AAB to the Play Console, the "App must support 16 KB
  memory page sizes" warning clears.

## iOS status

iOS is **not** subject to the 16 KB requirement (it's a Google Play / Android
policy). The iOS native project still targets the RN 0.73 layout
(`AppDelegate.h/.mm`, `main.m`). RN 0.85 switches iOS to a Swift `AppDelegate`
and rewrites the Xcode project, which is best completed on macOS. Until then,
the Android build is fully upgraded and the JS is shared.
