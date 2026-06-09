# Setup & Development

## Prerequisites

- **Node.js** >= 22.11.0
- **Yarn** (a `yarn.lock` is committed — prefer Yarn for consistency)
- React Native environment for your target platform — follow the official
  [Environment Setup](https://reactnative.dev/docs/environment-setup) guide:
  - **Android**: Android Studio, JDK 17, Android SDK (compile/target SDK 36,
    Build-Tools 36.0.0), **NDK 27.1.12297006**, an emulator or device.
    The Gradle wrapper auto-downloads Gradle 9.3.1 on first build.
  - **iOS** (macOS only): Xcode, CocoaPods

## Install

```bash
yarn install

# iOS only — install native pods
cd ios && pod install && cd ..
```

## Run

Start the Metro bundler in one terminal:

```bash
yarn start
```

Then launch the app from another terminal:

```bash
# Android
yarn android

# iOS
yarn ios
```

## Release build (signed AAB)

The Play Store release is built and signed through Android Studio (the wizard
signs with the selected keystore, overriding the `signingConfig` in
`android/app/build.gradle`):

1. Open the **`android/`** folder in Android Studio and let Gradle sync finish.
2. Bump `versionCode` (and usually `versionName`) in `android/app/build.gradle` —
   it must be **higher than the current live versionCode** on Play.
3. **Build → Generate Signed Bundle / APK → Android App Bundle**.
4. Select the upload keystore **`mywld`** (alias `mywld`) and enter its
   passwords, then build the **release** variant.
5. Output: `android/app/release/app-release.aab` → upload via Play Console
   → *Upload new version*.

> Signing keys live **outside** the repo (next to it): `mywld` is the current
> upload key; `upload_certificate.pem` is its public cert (only used to register
> the upload key with Play App Signing). The older `MyWoorld.jks` (2024) has been
> superseded. Keep keystores and passwords backed up and never commit them.

## Other Scripts

```bash
yarn lint   # ESLint
yarn test   # Jest
```

## Notes

- The app loads `https://mywoorld.com`, so a network connection is required.
- After changing native code or adding native dependencies, rebuild the app
  (re-run `yarn android` / `yarn ios`); a Metro reload is not enough.
