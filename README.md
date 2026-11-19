# RuneLite Mobile: alt touch controls

Builds **RuneLite Touch Test**, a test version of [RuneLite Mobile](https://github.com/zeruth/runelite-android)
with two additions:

1. **Plugin touch claims.** When a finger lands on something a plugin owns, the plugin sees the finger
   land and lift in real time. This is what lets on-screen keys stay held down while your thumb is on them.
   Other fingers keep working while one is claimed: taps reach the game and drags still move the camera.
2. **Touch Keys built in.** On-screen bubbles that act as keyboard keys (Shift, Space, numbers and more),
   with Hold and Toggle modes.

The app installs **beside** the official RuneLite app, with its own logins and settings. It doesn't send
crash reports to the official app's Firebase project.

## Install

Download the APK from the latest [release](../../releases/latest) and open it on your phone.
Once it's installed, the app checks this repository for updates the same way the official app checks
the official one.

## How updates work

The workflow in `.github/workflows/build.yml` is the auto-patcher. Every 6 hours it:

1. Looks up the latest official RuneLite Mobile release.
2. Stops if that version is already built here.
3. Otherwise checks out that version, applies the patches in `patches/`, builds the APK and publishes
   it as a release with the same version number.

You can also run it by hand from the **Actions** tab (**Build touch test** → **Run workflow**). Tick
**force** to rebuild a version that's already released. Changing a file in `patches/` rebuilds the
current version too.

If an official release changes the code the patches touch, the patches stop applying. The workflow
then fails and opens an issue here saying which version needs the patches updated.

## Setup: signing key

Android only installs an update if it's signed with the same key as the installed app. The workflow
reads that key from a repository secret:

- **Name:** `TOUCHTEST_KEYSTORE_BASE64`
- **Value:** the keystore file, base64-encoded (it's an Android debug keystore, so its passwords are the
  standard `android`)

Add it under **Settings** → **Secrets and variables** → **Actions** → **New repository secret**.
Without it, the workflow still builds the APK and attaches it to the run, but doesn't publish a release.

Keep the keystore private. Anyone who has it can sign an APK that installs as an update to your test app.

## Patches

| File | What it changes |
|---|---|
| `patches/0001-touch-claims.patch` | `AppAndroid.kt`: offers each touch on the game to RuneLite's mouse listeners first, and gives a claimed touch's moves and lift to the plugin. This is the change proposed to the RuneLite Mobile maintainers. |
| `patches/0002-touch-test-build.patch` | Test build only: the `.touchtest` app ID and "RuneLite Touch Test" name, no Crashlytics mapping upload or crash reporting, and Touch Keys bundled into the app. |

The workflow also points the app's update check at this repository's releases.
