# OrangeFox Recovery for Samsung Exynos 9820/9825 devices

Unofficial [OrangeFox Recovery](https://orangefox.tech) R12.0 builds for the Galaxy S10 and Note10
series and the Galaxy F62.

> [!WARNING]
> These are **test builds**. Only the Galaxy S10+ (beyond2lte) and Note10+ (d2s) have been tested
> on real hardware so far. Read the [requirements](#requirements) before flashing, and keep your current recovery
> image to go back.

**Downloads: [Releases](https://github.com/Exynos982x-OrangeFox/OrangeFox_Exynos982x/releases)**

## Devices

| Device | Codename | Models | Status |
| :-- | :-- | :-- | :-- |
| Galaxy S10e | beyond0lte | SM-G970F, SM-G970F/DS, SM-G970N | Needs testing |
| Galaxy S10 | beyond1lte | SM-G973F, SM-G973F/DS, SM-G973N | Needs testing |
| Galaxy S10+ | beyond2lte | SM-G975F, SM-G975F/DS, SM-G975N | Tested |
| Galaxy S10 5G | beyondx | SM-G977B, SM-G977N | Needs testing |
| Galaxy Note10 | d1 | SM-N970F, SM-N970F/DS | Needs testing |
| Galaxy Note10 5G | d1x | SM-N971N | Needs testing |
| Galaxy Note10+ | d2s | SM-N975F, SM-N975F/DS | Tested |
| Galaxy Note10+ 5G | d2x | SM-N976B, SM-N976N | Needs testing |
| Galaxy F62 | f62 | SM-E625F | Needs testing |

### Tested ROMs

Reported working with the `test-2026-10-31` build. *PIN tested* means /data was decrypted with a
lockscreen PIN and the PIN was still accepted in the ROM afterwards.

| Device | ROM | Android | PIN tested | Report |
| :-- | :-- | :-- | :-- | :-- |
| Galaxy S10+ (beyond2lte) | Evolution X 16.0 (2026-09-17) | 16 | Yes | Maintainer |
| Galaxy S10+ (beyond2lte) | Project Infinity-X 3.12 GApps (2026-07-10), custom kernel | 16 | Yes | Maintainer |
| Galaxy S10+ (beyond2lte) | Matrixx 11.4.0 Polar Vanilla | 15 | No (no lockscreen) | [#1](https://github.com/Exynos982x-OrangeFox/OrangeFox_Exynos982x/issues/1) |
| Galaxy S10+ (beyond2lte) | Matrixx 12.3 Binary Vanilla (2026-05-16) | 16 | Yes | [#1](https://github.com/Exynos982x-OrangeFox/OrangeFox_Exynos982x/issues/1#issuecomment-6796919518) |
| Galaxy Note10+ (d2s) | Evolution X 11.11 (2026-09-17) | 16 | Yes | [#2](https://github.com/Exynos982x-OrangeFox/OrangeFox_Exynos982x/issues/2) |
| Galaxy Note10+ (d2s) | LineageOS 23.2 nightly (2026-10-25) | 16 | Yes | [#3](https://github.com/Exynos982x-OrangeFox/OrangeFox_Exynos982x/issues/3) |

Each release provides, per device:

- `OrangeFox-R12.0-Unofficial-<codename>.img`, to flash with fastboot or from a recovery;
- `OrangeFox-R12.0-Unofficial-<codename>.tar`, to flash with Odin;
- a `.sha256` file next to each of them, to check your download.

**Flash only the file matching your codename.**

## Features

- /data decryption (FBE v2 with metadata encryption) with your lockscreen PIN, pattern or password.
  It uses the TEE, Keymaster and Gatekeeper of your installed ROM: no proprietary file is shipped,
  and the recovery keeps working when your ROM gets a new security patch.
- fastbootd, ADB, MTP.
- Backup and restore, zip and image flashing.
- microSD and USB-OTG storage.

## Requirements

- Unlocked bootloader, with verified boot (vbmeta) disabled. This is already the case if you
  installed LineageOS following its official instructions.
- A ROM built on the **LineageOS Exynos 9820 device trees**: LineageOS 21+, Evolution X, and most
  AOSP ROMs for these devices. These ROMs use the physical `system` partition as the super
  partition, and the recovery decrypts /data with the services of the ROM's vendor.
- **Not for stock One UI** or One UI based ROMs. One UI data can never be decrypted.

### Check that your ROM is compatible

With the ROM booted, USB debugging enabled, from a PC:

```sh
adb shell cat /proc/cmdline | grep -o 'androidboot.super_partition=[a-z]*'
adb shell ls /vendor/bin/hw/ | grep -E 'keymaster@4.0-service.samsung|gatekeeper@1.0-service'
adb shell ls /vendor/bin/tzdaemon /vendor/bin/tzts_daemon
```

You must get `androidboot.super_partition=system`, both services and both daemons. If not, do
not flash: decryption, and probably mounting the system, will not work.

So far this was only tested on LineageOS based ROMs. If you run PixelOS or another pure AOSP
ROM, your test report is especially welcome.

## Installation

Download the files for your device from the
[latest release](https://github.com/Exynos982x-OrangeFox/OrangeFox_Exynos982x/releases), with
their `.sha256` files, and check them:

```sh
sha256sum -c OrangeFox-R12.0-Unofficial-<codename>.img.sha256
```

Then pick one method.

### fastbootd (recommended)

1. Reboot to fastbootd: `adb reboot fastboot` from the ROM, or *Advanced → Enter fastboot* in
   LineageOS Recovery.
2. Flash and boot the recovery:

   ```sh
   fastboot flash recovery OrangeFox-R12.0-Unofficial-<codename>.img
   fastboot reboot recovery
   ```

### Odin

1. Boot to Download mode.
2. Put `OrangeFox-R12.0-Unofficial-<codename>.tar` in the **AP** slot, untick *Auto Reboot*, and
   press *Start*.
3. Leave Download mode straight into recovery with your device's recovery key combination, so that
   the ROM does not boot first.

### From an existing TWRP or OrangeFox

*Install → Install Image*, select the `.img`, then the *Recovery* partition.

## Testing and reporting

Please test and report with the
[test report form](https://github.com/Exynos982x-OrangeFox/OrangeFox_Exynos982x/issues/new?template=test_report.yml),
even if everything works: one report per device and ROM. What to test:

1. The recovery boots; display, touch and brightness work.
2. /data is decrypted with your lockscreen PIN, pattern or password (or without prompting if you
   have no lockscreen), and your files are visible in `/sdcard`.
3. **Back in the ROM, your lockscreen PIN, pattern or password is still accepted.**
4. ADB (`adb devices` shows the device in recovery) and MTP (files visible from a PC).
5. Flashing a zip (ROM update, Magisk, GApps...).
6. Backup and restore of `boot` and `data`.
7. fastbootd: *Reboot → Fastboot* from OrangeFox, then `fastboot devices`.
8. microSD and USB-OTG (if your device has them), flashlight, battery level.

For anything broken, open a
[bug report](https://github.com/Exynos982x-OrangeFox/OrangeFox_Exynos982x/issues/new?template=bug_report.yml)
with the logs. To collect them, while still in OrangeFox:

```sh
adb pull /tmp/recovery.log
adb logcat -d > logcat.txt
adb shell dmesg > dmesg.txt
```

## Known issues

### Lockscreen PIN refused in the ROM after using the recovery

Fixed in `test-2026-10-31`. The TEE daemon wrote its Gatekeeper data to `/efs/tee` as root and
without its SELinux label, and the ROM could not update it anymore. If it still happens, report it
with the output of these commands, run on the ROM with root:

```sh
ls -laZR /mnt/vendor/efs/tee | grep -E 'root|unlabeled'
dmesg | grep tzdaemon
```

To fix it, for each file listed by the first command, run as root:

```sh
chown system:system <file>
chcon u:object_r:tee_efs_file:s0 <file>
```

then reboot.

## Sources

- Common tree: [android_device_samsung_exynos9820-common](https://github.com/Exynos982x-OrangeFox/android_device_samsung_exynos9820-common)
- Device trees: [Exynos982x-OrangeFox](https://github.com/Exynos982x-OrangeFox), branch `fox_12.1`
  (each device tree's README has the build instructions)
- Kernels: prebuilt from the official LineageOS 23.2 recovery images, built from
  [android_kernel_samsung_exynos9820](https://github.com/LineageOS/android_kernel_samsung_exynos9820)
- [OrangeFox Recovery](https://gitlab.com/OrangeFox), branch `fox_12.1`

## Credits

- [OrangeFox Recovery](https://orangefox.tech) and [TeamWin](https://twrp.me)
- [LineageOS](https://github.com/LineageOS) for the platform configuration and the kernels
- [ExtremeXT](https://github.com/ExtremeXT/android_device_samsung_exynos9820) for the TWRP device tree
