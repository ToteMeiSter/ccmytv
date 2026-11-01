# TCL Smart TV G07 (G07_4K_GB_NF, AP firmware AR07, Realtek RTD2851A), Google TV 11

- `ro.product.model` = `Smart TV`, device `G07`, product `G07_4K_GB_NF`, client `TCL-AP-RTD51A-S1`
- Software `V8-R51AT01-LF1V098`, build `RP1A.200622.001` incremental `AR07`, patch 2023-10-05
- 2 GB RAM, `armeabi-v7a` only, SELinux enforcing, user build

Same hardware and package family as [`tcl-g07-google-tv`](../tcl-g07-google-tv/debloat.txt). This list is
a shorter cut for an owner who keeps firmware updates, voice and casting; the header of `debloat.txt`
lists what stays.

## What differs from the other G07 units

- **ADB.** `service.adb.tcp.port=5555` brings network debugging back after a reboot;
  `persist.adb.tcp.port` is empty on this firmware.
- **Home.** `cmd package set-home-activity` does not beat launcherx (HOME priority 2). After launcherx
  is disabled, setupwraith's `RecoveryActivity` takes Home (priority 1); after setupwraith, the other
  launcher does.
- **Boot.** AR07 re-enables `com.tcl.notereminder` and `com.tcl.usercenter` at boot, the same as the
  Brazil AR07 units. The other 18 packages stayed disabled after a reboot.
- **Chromecast.** `com.google.android.apps.mediashell` listens on 8008/8009 before and after the cut.
- **Input key.** `KEYCODE_TV_INPUT` opens the source panel (`com.tcl.tv`) over a third-party Home: TV,
  HDMI 1-3, AV.
- **Miracast.** `com.tcl.suspension` names `com.tcl.miracast.app.DUAL`, `MULTI` and `TV_MULTI`, but no
  package resolves those actions even with Miracast enabled, so disabling `com.tcl.miracast` does not
  break the panel.

## Known issues

- **Guide and Gear keys need launcherx.** This remote has no Netflix or YouTube key. Its Guide key
  (the "chart" icon, scan code 4001) goes through `com.tcl.tv/.service.GlobalKeyService`, which sends
  `KEYCODE_GUIDE`; the global key map (`dumpsys window policy`, mKeyMapping) delivers that only to
  `launcherx/.remotecontrol.GlobalKeyListener_Receiver`, before any app sees it. The Gear key (4077)
  opens the Google TV panel, which launcherx draws. With launcherx disabled both keys do nothing, and
  another launcher cannot remap them. Without root you cannot keep launcherx and drop only its Home:
  `pm disable --user 0 com.google.android.apps.tv.launcherx/.home.HomeActivity` fails with
  `SecurityException: Shell cannot change component state`, and an enabled launcherx takes Home back
  at once (priority 2). Pick one: the keys or a third-party Home. This owner kept the third-party Home;
  Live TV opens from the launcher's TV or input tile. The ivi, Okko, Kinopoisk and Media keys start
  their apps directly and keep working.
- **Third-party screensaver blocked by auto-start.** "Start now" and the idle timer do nothing for a
  sideloaded DreamService: the log shows `Unable to bind dream service` and TCL's boot manager refusing
  with `callee_does't_have_OP_AUTO_START_permission`. Allow it per package:
  `appops set <pkg> APP_AUTO_START allow` (undo: `... ignore`).
- **dreamx is optional.** The owner later re-enabled `com.google.android.apps.tv.dreamx` for the
  ambient screensaver (`dreamx/.service.Backdrop`, recent photos); it did not take Home. Drop it from
  `debloat.txt` if you want the same.

## Not checked over ADB

HDMI picture, the tuner's channels and casting from a phone need the real remote.
