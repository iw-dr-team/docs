# Crashlytics Issue Report - Android 3.2.2

Generated: 2026-08-05

Source: Firebase Crashlytics report API through Firebase MCP workflow for project `dancing-road`, Android app `com.amanotes.pamadancingroad` (`1:528923424982:android:46195f809d709f94`).

## Scope

- Platform: Android
- App version filter: `3.2.2`
- Version display names included: `3.2.2 (26072414)`, `3.2.2 (26072711)`, `3.2.2 (26072314)`
- Error type filter: `FATAL`
- Time window: default Crashlytics report window, `2026-07-29T00:00:00Z` to `2026-08-05T23:59:59Z`

Version distribution for fatal events in this window:

| Version display name | Fatal events | Track |
| --- | ---: | --- |
| `3.2.2 (26072414)` | 346 | production |
| `3.2.2 (26072711)` | 0 | n/a |
| `3.2.2 (26072314)` | 0 | n/a |

## Summary

- Total fatal events in Android `3.2.2`: 346
- Total fatal issue groups: 28
- Sum of impacted users per issue: 247. This is not de-duplicated across issues.
- Dominant issue: `bfb1fef5ce0c421e7149f9390db69604` (`libc.so / unknown`) with 297 events and 207 impacted users, accounting for about 86% of fatal events.
- Refresh for exact version `3.2.2 (26072414)` across all Crashlytics issue types returned 42,608 events.
- Issues over 50 events for exact version `3.2.2 (26072414)`: 5 total, including 4 `NON_FATAL` exception issues and 1 `FATAL` crash issue.

## Recommended Priority

1. `bfb1fef5ce0c421e7149f9390db69604` - native `libc.so` crash, 297 events / 207 users. This issue is marked regressed and dominates the build.
2. WebView `reasonPhrase can't be empty` family - `5e0a...`, `ecf4...`, `96dd...`, `213a...`; combined 14 events. The blamed frames point to obfuscated third-party/app code creating `WebResourceResponse`.
3. Native `libil2cpp.so` crashes - `c2c...` and `8f0...`; combined 7 events. Sample breadcrumbs point at ad/gameplay contexts.
4. Memory pressure/OOM issues - `c615...`, `83ae...`, `a5e...`, plus `c261...`; combined 7 events. These affect lower-end devices in samples.
5. Android system death issues - `23ef...`, `3080...`, `3629...`; combined 5 events. Likely lower actionability unless correlated with lifecycle/service handling.

## All Issue Types Over 50 Events

This section uses the exact version filter `3.2.2 (26072414)` and does not restrict `issueErrorTypes`, so it includes `FATAL`, `NON_FATAL`, and `ANR` if present. Threshold: `eventsCount > 50`.

Time window: `2026-07-29T00:00:00Z` to `2026-08-05T23:59:59Z`.

Total Crashlytics events for `3.2.2 (26072414)` in this window: 42,608.

| # | Issue ID | Type | Title | Exception / Subtitle | Events | Users | Sessions | State |
| ---: | --- | --- | --- | --- | ---: | ---: | ---: | --- |
| 1 | `f76aae02eefa4b673a6e8c15bfc402a9` | NON_FATAL | `DR.SeasonalTheme.SeasonalThemeSpriteChanger - DR.SeasonalTheme.SeasonalThemeSpriteChanger.Awake` | `java.lang.Exception - ArgumentOutOfRangeException : Index was out of range. Must be non-negative and less than the size of the collection. Parameter name: index` | 40,785 | 8,221 | 8,843 | OPEN |
| 2 | `bfb1fef5ce0c421e7149f9390db69604` | FATAL | `libc.so` | `unknown` | 303 | 210 | 297 | OPEN |
| 3 | `fa7806650f7c4cce2d665d3c20e29c41` | NON_FATAL | `InfiniteRun.get_PositionSpline` | `java.lang.Exception - NullReferenceException : Object reference not set to an instance of an object.` | 172 | 27 | 27 | OPEN |
| 4 | `68e63fe01dc4879e05d69680e37d8eb8` | NON_FATAL | `UserPermissions+<DelayShowFirstPermissionsGroup>d__11.MoveNext` | `java.lang.Exception - AndroidJavaException : java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState` | 167 | 160 | 166 | OPEN |
| 5 | `fe1bb2fa51d5852e6790eef388c6a83f` | NON_FATAL | `DR.GameResult.ButtonCurencyBase - DR.GameResult.ButtonCurencyBase.set_Interactable` | `java.lang.Exception - NullReferenceException : Object reference not set to an instance of an object.` | 86 | 20 | 40 | OPEN |

## Non-Fatal Exceptions Over 50 Events

| # | Issue ID | Title | Exception | Events | Users | Notes |
| ---: | --- | --- | --- | ---: | ---: | --- |
| 1 | `f76aae02eefa4b673a6e8c15bfc402a9` | `DR.SeasonalTheme.SeasonalThemeSpriteChanger.Awake` | `ArgumentOutOfRangeException: Index was out of range. Parameter name: index` | 40,785 | 8,221 | This issue alone accounts for about 95.7% of all events on `3.2.2 (26072414)`. |
| 2 | `fa7806650f7c4cce2d665d3c20e29c41` | `InfiniteRun.get_PositionSpline` | `NullReferenceException: Object reference not set to an instance of an object.` | 172 | 27 | Gameplay/path access issue. |
| 3 | `68e63fe01dc4879e05d69680e37d8eb8` | `UserPermissions.DelayShowFirstPermissionsGroup.MoveNext` | `AndroidJavaException: java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState` | 167 | 160 | Android lifecycle timing issue while showing permission UI. |
| 4 | `fe1bb2fa51d5852e6790eef388c6a83f` | `DR.GameResult.ButtonCurencyBase.set_Interactable` | `NullReferenceException: Object reference not set to an instance of an object.` | 86 | 20 | Game result UI state/null reference issue. |

## Fatal Issues

| # | Issue ID | Title | Exception / Subtitle | Events | Users | State | Signal |
| ---: | --- | --- | --- | ---: | ---: | --- | --- |
| 1 | `bfb1fef5ce0c421e7149f9390db69604` | `libc.so` | `unknown` | 297 | 207 | OPEN | REGRESSED |
| 2 | `5e0a0977aa88b2a9e1d4270f74e7af7a` | `SourceFile - xq.a` | `java.lang.IllegalArgumentException - reasonPhrase can't be empty.` | 8 | 4 | OPEN |  |
| 3 | `c2cfc3b597373a8cc2e4228c6d17cff8` | `[libil2cpp.so]` |  | 5 | 5 | OPEN |  |
| 4 | `c6150e254b0e0ff096fe756299684825` | `com.unity3d.player.v.a` | `java.lang.OutOfMemoryError` | 4 | 3 | OPEN |  |
| 5 | `23ef0b2efb89a5e8e63dd43d82aee5f6` | `android.app.ActivityThread.handleStopService` | `android.os.DeadSystemException` | 3 | 1 | OPEN |  |
| 6 | `50d0084b4d45d9550abeb52f23b080bc` | `chromium-SystemWebViewGoogle6432.aab-stable-772713803 - org.chromium.android_webview.common.AwResource.getConfigKeySystemUuidMapping` | `android.content.res.Resources$NotFoundException - String array resource ID #<address>` | 3 | 3 | OPEN |  |
| 7 | `ecf4e80177fda3fc1c3392d21839145c` | `SourceFile - xy.a` | `java.lang.IllegalArgumentException - reasonPhrase can't be empty.` | 3 | 2 | OPEN |  |
| 8 | `8f0be5b432c60dfc5bf0164380f5b4d5` | `[libil2cpp.so]` | `SIGSEGV` | 2 | 2 | OPEN |  |
| 9 | `96ddfe28b68778a8ce0644ccf7721f8d` | `SourceFile - xf.a` | `java.lang.IllegalArgumentException - reasonPhrase can't be empty.` | 2 | 1 | OPEN |  |
| 10 | `02b34f5f5c474a1c2411a7ad739d54a1` | `android.provider.Settings$NameValueCache.getStringForUser` | `java.lang.SecurityException - Settings key: <screenshot_pointer> is not readable...` | 1 | 1 | OPEN | REPETITIVE |
| 11 | `1f3b95fa768587c2b1e38ea32774b043` | `com.android.billingclient:billing@@8.0.0 - com.google.android.gms.internal.play_billing.zzbg.zze` | `java.lang.IllegalStateException - This stopwatch is already running.` | 1 | 1 | OPEN |  |
| 12 | `213a047ac356efcd2de91dfd8b39400b` | `PG - yk.a` | `java.lang.IllegalArgumentException - reasonPhrase can't be empty.` | 1 | 1 | OPEN |  |
| 13 | `24a1d65665120f435202bbb03ce4b681` | `[libc.so]` |  | 1 | 1 | OPEN |  |
| 14 | `30805a6146c0b33feed724acdf88553d` | `android.app.ActivityThread.handleCreateService` | `android.os.DeadSystemException` | 1 | 1 | OPEN | EARLY |
| 15 | `36293788e50e498455ee6f557bd0a311` | `android.app.ActivityThread.handleUnbindService` | `android.os.DeadSystemException` | 1 | 1 | OPEN |  |
| 16 | `512fff8cdddc3fdc15e66fbd37824ad0` | `com.unity3d.player.UnityPlayerActivity.onResume` | `java.lang.IllegalArgumentException` | 1 | 1 | OPEN |  |
| 17 | `533ba16ef702bb171bab13db83b4f2c3` | `[libmain.so] __ThumbV7PILongThunk_free` |  | 1 | 1 | OPEN |  |
| 18 | `60ac1683ca1187469a092aa4a2bc3795` | `[libmonochrome.so]` |  | 1 | 1 | OPEN |  |
| 19 | `83ae1c9435245cd228f95f19957303e3` | `java.lang.Thread.nativeCreate` | `java.lang.OutOfMemoryError` | 1 | 1 | OPEN |  |
| 20 | `846f220a82a98489a98dc47bf9427ec2` | `com.anzu.sdk.Anzu$1$2.onSharedPreferenceChanged` | `java.lang.NullPointerException - Attempt to invoke virtual method 'boolean java.lang.String.equals(java.lang.Object)' on a null object reference` | 1 | 1 | OPEN |  |
| 21 | `85ad673779760535aa678710d67cbaba` | `Thread.java` | `java.lang.Thread.nativeCreate` | 1 | 1 | OPEN |  |
| 22 | `974b1e843423a4ae2e0ad840e29ce106` | `[libGLESv1_CM.so]` | `SIGSEGV` | 1 | 1 | OPEN |  |
| 23 | `a5e73b3c246cc42e346c92583eb1947c` | `com.google.firebase.database.tubesock.WebSocket.connect` | `java.lang.OutOfMemoryError` | 1 | 1 | OPEN |  |
| 24 | `ad07bbcdf9a7b27ad827322c5fecad38` | `com.facebook.unity.FBUnityGamingServicesFriendFinderActivity.onCreate` | `java.lang.NullPointerException - Attempt to invoke virtual method 'java.lang.String android.os.Bundle.getString(java.lang.String)' on a null object reference` | 1 | 1 | OPEN |  |
| 25 | `af29592d1dbe462fc17f0c5717e8b371` | `libart.so` | `unknown` | 1 | 1 | OPEN |  |
| 26 | `bb0501a4116cbf19d0cf644dc81922b3` | `com.facebook.unity.FBUnityLoginActivity.onCreate` | `java.lang.NullPointerException - Attempt to invoke virtual method 'int com.facebook.unity.FBUnityLoginActivity$LoginType.ordinal()' on a null object reference` | 1 | 1 | OPEN |  |
| 27 | `c2615155ca8802711876867eb2139570` | `android.database.CursorWindow.nativeCreate` | `android.database.CursorWindowAllocationException - Could not allocate CursorWindow '/data/user/0/com.amanotes.pamadancingroad/databases/metrics-db' of size 2097152 due to error -12.` | 1 | 1 | OPEN |  |
| 28 | `e9d0974aa5c6b2e2646f314b54a9cae1` | `[libufwriter.so]` | `SIGSEGV` | 1 | 1 | OPEN |  |

## Sample Events For Top Issues

The events below were fetched with the same `FATAL` and `3.2.2` version filters. This avoids the `topIssues.sampleEvent` caveat where a sample event can come from a different app version.

### 1. `bfb1fef5ce0c421e7149f9390db69604` - `libc.so`

- Event time: `2026-08-05T09:27:58Z`
- Version: `3.2.2 (26072414)`
- Device: `ZTE (K87CA)`, Android `10`, ARMV7
- Recent breadcrumbs: `bannerads_request`, `bannerads_show`, `screen_view: InMobiAdActivity`, `ad_impression` from `inmobi`, `ad_impression_ama`
- Crashed/blamed native frames:

```text
libwebviewchromium.so @ 38688176
libwebviewchromium.so @ 5237718
libwebviewchromium.so @ 70754124
libunity.so @ 4395619
libil2cpp.so @ 16709229
libc.so @ 632088 (blamed)
libutils.so @ 67589
libart.so @ 847876
```

### 2. `5e0a0977aa88b2a9e1d4270f74e7af7a` - `reasonPhrase can't be empty`

- Event time: `2026-08-04T23:41:56Z`
- Version: `3.2.2 (26072414)`
- Device: `samsung (SM-J327W)`, Android `8.1.0`, ARMV7
- Root exception: `java.lang.IllegalArgumentException: reasonPhrase can't be empty.`
- Blamed frame: `SourceFile xq.a:292`
- Recent breadcrumbs: `videoads_request`, `sub_offer_close`, `fullads_request_success`

```text
android.webkit.WebResourceResponse.setStatusCodeAndReasonPhrase(WebResourceResponse.java:138)
android.webkit.WebResourceResponse.<init>(WebResourceResponse.java:76)
SourceFile xq.a:292 (blamed)
SourceFile alW.handleMessage:67
android.os.Handler.dispatchMessage(Handler.java:106)
android.os.Looper.loop(Looper.java:164)
android.app.ActivityThread.main(ActivityThread.java:7000)
```

### 3. `c2cfc3b597373a8cc2e4228c6d17cff8` - `[libil2cpp.so]`

- Event time: `2026-08-04T22:33:48Z`
- Version: `3.2.2 (26072414)`
- Device: `samsung (SM-A022F)`, Android `11`, ARMV7
- Recent breadcrumbs include tutorial/gameplay flow and `fullads_request_success`.
- Blamed native frames:

```text
libc.so @ 633804
unknown @ 1785882823
unknown @ 1768700524
unknown @ 1834967904
unknown @ 1802073706
libil2cpp.so @ 25250912 (blamed)
```

### 4. `c6150e254b0e0ff096fe756299684825` - `java.lang.OutOfMemoryError`

- Event time: `2026-08-04T20:56:00Z`
- Version: `3.2.2 (26072414)`
- Device: `samsung (SM-A022F)`, Android `11`, ARMV7
- Root exception: `java.lang.OutOfMemoryError`
- Blamed frame: `com.unity3d.player.J.a:10`
- Recent breadcrumbs: `album_show`, `bannerads_reload`, `bannerads_request_failed`, `song_preview`

```text
android.graphics.Bitmap.nativeCreate(Bitmap.java)
android.graphics.Bitmap.createBitmap(Bitmap.java:1163)
android.graphics.Bitmap.createBitmap(Bitmap.java:1117)
android.graphics.Bitmap.createBitmap(Bitmap.java:1067)
android.graphics.Bitmap.createBitmap(Bitmap.java:1028)
com.unity3d.player.J.a:10 (blamed)
com.unity3d.player.h0.surfaceDestroyed:36
android.view.SurfaceView.notifySurfaceDestroyed(SurfaceView.java:2025)
```

### 5. `23ef0b2efb89a5e8e63dd43d82aee5f6` - `android.os.DeadSystemException`

- Event time: `2026-07-30T02:30:22Z`
- Version: `3.2.2 (26072414)`
- Device: `Android (Android)`, Android `14`, ARM64
- Root exception: `java.lang.RuntimeException: Unable to stop service ... JobInfoSchedulerService ... android.os.DeadSystemException`
- Crashed thread: `main`

```text
android.app.ActivityThread.handleStopService(ActivityThread.java:4984)
android.app.ActivityThread$H.handleMessage(ActivityThread.java:2389)
android.os.Handler.dispatchMessage(Handler.java:106)
android.os.Looper.loopOnce(Looper.java:205)
android.os.Looper.loop(Looper.java:294)
android.app.ActivityThread.main(ActivityThread.java:8440)
```

## Notes

- The report uses Crashlytics `FATAL` events only. ANR and non-fatal exceptions are excluded.
- `ProjectSettings/ProjectSettings.asset` currently shows local Unity `bundleVersion: 3.2.4`; this report is specifically filtered to Crashlytics app version `3.2.2`.
- Firebase MCP Crashlytics tools were not directly exposed in this session because the MCP availability detector looks for native Android Gradle Crashlytics signals, while this Unity project declares Crashlytics through Unity/EDM dependencies. The data above was fetched through the same Firebase Crashlytics report/event API used by the Firebase MCP Crashlytics implementation after reading the MCP Crashlytics report guide.
