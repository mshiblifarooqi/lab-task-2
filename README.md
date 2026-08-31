# Al Farooq Optics — Android Build Handoff Notes

**Repo:** https://github.com/muhammadshibli23459/Al-Farooq-inventory-Android
**Server:** AWS EC2 (Ubuntu), instance `i-042789f9b06ac3080` ("Optics 10/08/2026"), region `eu-north-1b`, type `t3.micro`
**Project path on server:** `/home/ubuntu/Andriod/Al-Farooq-inventory-Android`

---

## What this project is
- **Not** a native Android Studio project — it's an **Expo / React Native** app (SDK 51).
- Uses classic React Navigation (`App.js` → `src/navigation/AppNavigator.js` → `src/screens/...`), **not** expo-router.
- Local SQLite DB (`expo-sqlite`), Axios for API calls, AsyncStorage for local storage.

---

## Server setup done
- Node.js installed via **nvm**, currently on **Node 20** (`nvm use 20`) — Node 22 caused module-resolution errors with Expo SDK 51, so stick to Node 20.
- Java 17 (`openjdk-17-jdk`) installed.
- Android SDK installed manually under `~/android-sdk` (cmdline-tools, platform-tools, `platforms;android-34`, `build-tools;34.0.0`). `ANDROID_HOME` exported in `~/.bashrc`.
- **EBS volume resized twice**: 8GB → 16GB → **35GB** (via AWS Console → Volumes → Modify Volume), then on the instance:
  ```bash
  sudo growpart /dev/nvme0n1 1
  sudo resize2fs /dev/nvme0n1p1
  ```
- **Swap file** added and increased to **4GB** (`/swapfile` + `/swapfile2`, both in `/etc/fstab`) — needed because the instance only has **908MB RAM**, which is very tight for Android/Gradle builds.
- `android/gradle.properties` tuned for low memory:
  ```
  org.gradle.jvmargs=-Xmx1280m -XX:MaxMetaspaceSize=512m
  org.gradle.daemon=false
  org.gradle.parallel=false
  ```
  Builds are run with `--no-daemon` for stability on this small instance.

---

## Missing assets fixed
`app.json` referenced `assets/icon.png`, `assets/splash.png`, `assets/adaptive-icon.png` but the `assets/` folder didn't exist in the repo. Placeholder PNGs (dark theme, "AF" logo) were generated and uploaded into `assets/`. **These are placeholders — swap in the real logo/splash whenever the client provides one, same filenames.**

---

## Dependency version issues fixed (via `package.json` → `"overrides"`)
Several Expo-family packages had resolved to versions way newer than SDK 51 expects (likely due to loose ranges pulled in through `expo-router`), which broke Gradle plugin resolution (`expo-module-gradle-plugin not found`, `unknown property 'release'` errors). Fixed by pinning:
```json
"overrides": {
  "expo-constants": "~16.0.2",
  "expo-linking": "~6.3.1"
}
```
Also ran `npx expo install --fix` (aligned `react-native` → 0.74.5, `react-native-safe-area-context` → 4.10.5) and `npx expo install react-native-reanimated` (added because `babel.config.js` requires the `react-native-reanimated/plugin` babel plugin, which wasn't in `package.json`).

If more `expo-module-gradle-plugin not found` errors show up for other packages, check that package's version against SDK 51's expected versions (see `https://raw.githubusercontent.com/expo/expo/sdk-51/packages/expo/bundledNativeModules.json`) and add an override the same way.

---

## Root cause of the app crashing on the phone ("Al Farooq Optics keeps stopping")
`package.json` had:
```json
"main": "expo-router/entry"
```
This makes the app try to boot via **expo-router**, which expects a file-based `app/` directory — but this project doesn't have one (it uses `App.js` + classic navigation). Result: app loads the splash screen, then crashes immediately.

**Fix applied:**
```json
"main": "node_modules/expo/AppEntry.js"
```
And removed `expo-router` from `app.json`'s `plugins` array (now `"plugins": []`).

After this fix + a clean prebuild + rebuild, a new `app-release.apk` was built successfully (verified bundling used `node_modules/expo/AppEntry.js`, not `expo-router/entry.js`, in the build log). **This APK has not yet been confirmed working on the phone** — that's the next thing to verify.

---

## Standard build commands (once dependencies are healthy)
```bash
cd ~/Andriod/Al-Farooq-inventory-Android
rm -rf android
npx expo prebuild --platform android --clean
cd android
./gradlew assembleRelease --no-daemon
```
Output APK: `android/app/build/outputs/apk/debug/app-debug.apk` or `.../release/app-release.apk`.

**Note on debug vs release APK:** the debug APK needs a live Metro dev server running and reachable from the phone (won't work standalone off-network) — that's expected, not a bug. Use the **release** APK for a standalone installable app; it bundles the JS inside the APK.

⏱️ **Build time expectation on this instance:** due to only 908MB RAM (even with 4GB swap), a full release build has taken **30 minutes to over 1.5 hours**. This is normal for this instance size — not a sign of a stuck build. Confirm a build is still alive (not frozen) with `top` — look for a `java` process actively using CPU%.

---

## Current status / next step
User asked to wipe everything and rebuild clean from scratch:
```bash
cd ~/Andriod/Al-Farooq-inventory-Android
rm -rf android
rm -rf node_modules package-lock.json
npm install
npx expo prebuild --platform android --clean
cd android
./gradlew assembleRelease --no-daemon
```
This was in progress / about to run when these notes were written. Once it completes, grab the APK from:
```
~/Andriod/Al-Farooq-inventory-Android/android/app/build/outputs/apk/release/app-release.apk
```
uninstall any old version on the phone, install this one, and confirm the crash is gone (splash screen should be followed by the actual login/app screen now, not a crash).

---

## Useful troubleshooting commands
```bash
df -h                 # disk space check (has run out multiple times — watch this)
free -h               # RAM/swap check
top                    # confirm a running build isn't frozen (look for java process CPU%)
node -v                # should be v20.x (via nvm) — NOT v22
cat package.json | grep '"main"'   # should say node_modules/expo/AppEntry.js
```

## Long-term recommendation
This t3.micro (908MB RAM) instance is undersized for Android/Expo builds — every build is slow and disk/RAM has run out repeatedly. If builds need to happen regularly, consider temporarily resizing to a bigger instance type (e.g. t3.medium, 4GB RAM) for build sessions — would cut build time from ~1 hour down to ~5-10 minutes.
