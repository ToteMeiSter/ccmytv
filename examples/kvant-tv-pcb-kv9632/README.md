# Kvant TV, TV_PCB_KV9632 (MediaTek m7332, Ozon firmware), Android TV 11

- `ro.product.model` = `TV_PCB_KV9632`, `ro.product.manufacturer` = `Kvant_Ltd`
- Fingerprint `MediaTek/m7332_eu/m7332:11/ozon/1313`, user build, SELinux enforcing
- About 1.4 GB RAM, `armeabi-v7a` only, 1920x1080
- No Google Play, no Play Services, no Google TV / tvlauncher

The vendor layer is OneWay (`com.oneway.*`) with Ozon overlays (`com.ozon.*` RROs). The partner and
ad apps already ship disabled (list in the `debloat.txt` header), so the list itself is short.

## ADB

- Port 5555 is open with network debugging on. `ro.adb.secure=1`: the first `adb connect` reports
  `unauthorized` until the "Allow USB debugging?" prompt is accepted on the TV.
- `persist.adb.tcp.port=5555` survives reboots: after both test reboots 5555 answered by itself.
- `adb exec-out screencap -p` puts two vendor log lines (`Init wrapper sys mutex successful`,
  `open mma dev failed`) in front of the PNG, so the file is not a valid image. Capture to a file on the
  TV (`screencap -p /sdcard/s.png`) and pull it instead.

## Kept, and why (checked on this TV)

| Package | Why it stays | Evidence |
|---|---|---|
| `com.oneway.tv` | Every TvInputService: analog, DVB-T/C/S, HDMI, composite, component | `pm query-services --components -a android.media.tv.TvInputService` |
| `com.oneway.tv.live` | Live TV ("Эфирное ТВ"), the only Live TV app | LEANBACK_LAUNCHER entry `.playback.TvPlaybackActivity` |
| `com.oneway.tv.inputchooser` | Input menu. The remote's Input key (`KEYCODE_TV_INPUT`) opens it over any launcher | `mCurrentFocus` = `TvInputChooserActivity` after the key, with a third-party Home |
| `com.oneway.tv.core.services` | Persistent system service (uid 1000) under the shell | `flags=[ SYSTEM PERSISTENT ]` |
| `com.oneway.updater` | Firmware updates (OTA) | owner's choice |
| `com.oneway.tv.onetimeinit` | First-boot setup; not proven safe to disable | - |
| `com.mediatek.tv.service`, `com.mediatek.loadingequalizer` | TV middleware (the core guard refuses `com.mediatek.tv.service`) | - |
| `com.ecloud.eshare.server`, `com.ecloud.emedia` | Screen sharing (EShare) and the DLNA renderer that phone apps cast to. EShare's code refers to EDLNA and EAirPlay, so disabling one may take the other with it | strings in `EShareServer.apk` |
| `com.oneway.tv.youtube` | YouTube client (OneTube). The vendor `dialserver` (DIAL, port 56789) is what phones use to send YouTube to the TV | `/proc/net/tcp6` listeners |
| `com.android.tv.settings` | Settings (`android.settings.SETTINGS` handler) | core guard |
| `com.android.inputmethod.leanback` | The only keyboard | `default_input_method` |
| `com.oneway.dreams` | Screensaver set in `screensaver_components` | - |
| `ru.vk.store.tv` | RuStore, the only app store on the set | - |

No Google Cast receiver exists on this firmware: no `mediashell`, ports 8008/8009 refuse, and
`dumpsys media_router` lists no providers. "Casting" from a phone goes through DLNA (EDLNA), DIAL
(YouTube) or EShare.

## Third-party launcher

Sideloading works (`adb install`). A launcher that declares HOME takes over with
`cmd package set-home-activity <pkg>/<activity>`; `resolve-activity` and
`dumpsys activity activities | grep mFocusedApp` both named it, before and after reboots, and it was
on screen at power-on (`wm_set_resumed_activity` goes FallbackHome, then the new launcher). Then
`com.oneway.tv.home` can be disabled. Live TV is not opened at power-on by the firmware.

The Wi-Fi link comes up a few seconds after Home starts at boot: a launcher that fetches anything
over the network at start should retry.

The set ships SELinux enforcing, so an ordinary app cannot read `/proc/stat` (`avc: denied { getattr }`
for `proc_stat`): a launcher's CPU meter has nothing to show here.

## Wallpapers

No pictures are kept in this folder (licences and size). A local photo slideshow works from shared
storage: push your own folder with `adb push <folder>/. /sdcard/Pictures/Wallpapers/` and, on Android 11,
grant the launcher `android.permission.READ_EXTERNAL_STORAGE` with `pm grant` if it reads that folder.

## Known issues

### Live TV crashes when an HDMI input has no signal (firmware defect)

This is a firmware bug, not a result of the cleanup. Do not re-enable packages to fix it.

- The Input menu (`com.oneway.tv.inputchooser`, the remote's Input key) opens an input with VIEW
  `content://android.media.tv/passthrough/com.oneway.tv%2F.hdmi.HDMIInputService%2FHW<N>` in
  `com.oneway.tv.live/.playback.InputPlaybackActivity`.
- If that HDMI port has no signal (`dumpsys tv_input` shows `state: 1`, `CONNECTED_STANDBY`),
  Live TV ("Эфирное ТВ") crashes: `NullPointerException` in
  `com.oneway.tv.live.view.PlaybackView$b.onVideoUnavailable` (its `TvPlaybackActivity` field is
  not initialised).
- Re-enabling the stock launcher `com.oneway.tv.home` does not stop the crash (checked). No link to
  disabling the launcher was found.
- The same URI opened with the explicit component `TvPlaybackActivity` does not crash:
  `am start -a android.intent.action.VIEW -d "<uri>" -n com.oneway.tv.live/.playback.TvPlaybackActivity`.
- Workaround: use a launcher that opens inputs through `com.oneway.tv.live/.playback.TvPlaybackActivity`.

### Arc screensaver cannot be selected in Settings (Arc before 1.2.0-mgd.12)

Before build 4138, Arc's `screensaver.xml` declared a `settingsActivity` that does not exist, and
TvSettings crashed before it saved the choice. From 4138 the choice works. On older builds set it
over adb:
`adb shell settings put secure screensaver_components ru.mgdatv.arc/com.omeda.arc.ClockScreensaverService`
and `adb shell settings put secure screensaver_enabled 1`. Check with
`adb shell dumpsys dreams | grep mCurrentDreamName` while the screensaver runs.
