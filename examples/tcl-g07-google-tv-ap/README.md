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

## Not checked over ADB

HDMI picture, the tuner's channels, casting from a phone and the remote's Gear, Netflix and YouTube
buttons (see `tcl-g07-google-tv` for those keys) need the real remote.
