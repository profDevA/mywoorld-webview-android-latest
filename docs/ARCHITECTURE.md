# Architecture

MyWoorld is a thin native shell around a web app. The native layer handles
navigation, OS permissions, and hosting the WebView; the actual UI is served by
`mywoorld.com`.

Built on **React Native 0.85.3 / React 19** with the **New Architecture**
(Fabric/TurboModules) enabled on Android. See
[`16KB_UPGRADE.md`](./16KB_UPGRADE.md) for the upgrade history and the Google
Play 16 KB page-size fix.

## Runtime Diagram

```
index.js
  └─ AppRegistry.registerComponent(App)
        └─ src/App.js  (NavigationContainer → Stack.Navigator)
              ├─ "Home"     → src/screens/Home.js
              │                 ├─ requests CAMERA + MICROPHONE permissions
              │                 └─ <WebView uri="https://mywoorld.com" />
              └─ "Welcome"  → src/screens/Welcome.js  (placeholder)
```

## Layers

### Navigation (`src/App.js`)
A stack navigator with headers hidden. `Home` is the entry route. New screens
are added as `Stack.Screen` children here.

### Web content (`src/screens/Home.js`)
The core of the app. A full-screen `WebView` points at `https://mywoorld.com`.
All product features live on the web side.

### Permissions (`src/screens/Home.js`)
On mount, the screen checks and requests camera and microphone access via
`react-native-permissions`. These are intended to let the embedded site use
media features (e.g. video/audio).

## Native Configuration

### Android
- `android/app/src/main/AndroidManifest.xml` declares `INTERNET`, `CAMERA`,
  and `RECORD_AUDIO` permissions.
- Toolchain: compile/target SDK 36, Build-Tools 36.0.0, **NDK 27.1.12297006**,
  Kotlin 2.1.20, Gradle 9.3.1, AGP via the React Native Gradle plugin.
- `newArchEnabled=true` (New Architecture), `hermesEnabled=true`,
  `edgeToEdgeEnabled=false`. Autolinking is plugin-based
  (`settings.gradle` + `autolinkLibrariesWithApp()`).
- The NDK 27 build produces 16 KB-aligned native libraries (Play requirement).

### iOS
- `ios/MyWoorld/Info.plist`: App Transport Security enabled
  (`NSAllowsArbitraryLoads` = false). Add usage description keys here for any
  runtime permission the app requests.
- **Note:** the iOS native project is still on the RN 0.73 layout and has not
  been migrated to RN 0.85 (Swift `AppDelegate` + new Xcode project). Complete
  this on macOS before any App Store build.

## Known Gaps / Follow-ups

These are documented for awareness (not yet addressed):

1. **iOS usage descriptions missing** — `Info.plist` lacks
   `NSCameraUsageDescription` / `NSMicrophoneUsageDescription`, which iOS
   requires before requesting those permissions (otherwise the app crashes).
2. **Android runtime permissions not requested** — `Home.js` only handles
   `PERMISSIONS.IOS.*`; Android 6+ needs runtime requests too.
3. **WebView media access not wired** — granting OS permissions alone does not
   let the embedded site use `getUserMedia`; the `WebView` needs media-permission
   props/handlers.
4. **`Welcome` screen is dead code** — registered but unreachable.
