# MyWoorld — Project Overview

MyWoorld is a **React Native** mobile application that wraps the website
[`https://mywoorld.com`](https://mywoorld.com) inside a WebView and packages it
as an installable **iOS** and **Android** app.

## Purpose

Deliver the MyWoorld web experience as a native app, with native-level access to
device features (camera and microphone) that the embedded site can use.

## Tech Stack

| Area | Library | Version |
| --- | --- | --- |
| Framework | `react-native` | 0.85.3 |
| UI runtime | `react` | 19.2.3 |
| Navigation | `@react-navigation/native` + `@react-navigation/stack` | 7.x |
| Native screens | `react-native-screens` | 4.x |
| Web content | `react-native-webview` | 13.x |
| Permissions | `react-native-permissions` | 5.x |
| Gestures | `react-native-gesture-handler` | 3.x |
| Safe area | `react-native-safe-area-context` | 5.x |

Requires **Node >= 22.11.0**. The New Architecture (Fabric/TurboModules) is
enabled on Android. See [`16KB_UPGRADE.md`](./16KB_UPGRADE.md) for the upgrade
history and the Google Play 16 KB page-size fix, and
[`SETUP.md`](./SETUP.md#release-build-signed-aab) for the signed-release flow.

Current Android release: **versionCode 4 / versionName 2.2**
(`android/app/build.gradle`).

## App Flow

1. **`index.js`** registers the root `App` component using the name from `app.json`.
2. **`src/App.js`** sets up a `NavigationContainer` with a stack navigator
   (headers hidden). The initial route is **`Home`**.
3. **`src/screens/Home.js`** requests camera/microphone permissions on mount and
   renders the `mywoorld.com` site full-screen in a `WebView`.
4. **`src/screens/Welcome.js`** is a placeholder screen — registered in the
   navigator but not currently reachable.

## Directory Layout

```
MyWoorld/
├── index.js                 # App entry — registers root component
├── app.json                 # App name / display name
├── src/
│   ├── App.js               # Navigation container + stack
│   └── screens/
│       ├── Home.js          # WebView + permissions (core screen)
│       └── Welcome.js       # Placeholder screen
├── android/                 # Native Android project
├── ios/                     # Native iOS project
├── __tests__/               # Jest tests
└── docs/                    # Project documentation
```

## Related Docs

- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — how the pieces fit together
- [`SETUP.md`](./SETUP.md) — environment setup and run instructions
