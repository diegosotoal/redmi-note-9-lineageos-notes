# LineageOS 24.0 on the Xiaomi Redmi Note 9 (merlinx)

This repository records one successful installation of LineageOS 24.0 on a Xiaomi Redmi Note 9, completed on **2026-10-20**. The phone started on Ubuntu Touch. It was returned to Xiaomi stock Android 12 firmware first, then Lineage Recovery and LineageOS were installed.

This is an installation log for the exact versions listed below. Device requirements and download links can change; read the current [official LineageOS installation guide for merlinx](https://wiki.lineageos.org/devices/merlinx/install/) before repeating the process.

## Result and device

- Device: Xiaomi Redmi Note 9, LineageOS device name **merlinx**
- Fastboot product reported by the phone: `merlin`
- Initial OS: Ubuntu Touch
- Initial vendor build properties reported in Ubuntu Touch: `V12.5.4.0.RJOMIXM` and Android `11`
- Bootloader: already unlocked (`fastboot getvar unlocked` returned `yes`)
- Final OS: official LineageOS 24.0 nightly; first boot reached the LineageOS welcome screen
- Google apps: not installed

**Data loss:** returning to stock and formatting data erased the phone. Back up anything important before following these steps.

## Exact files and versions used

### Stock Android 12 firmware

The phone needed Android 12 stock firmware before LineageOS. The Global firmware package used was:

- Version: `V13.0.2.0.SJOMIXM` (MIUI 13, Android 12)
- Archive: [merlin_global_images_V13.0.2.0.SJOMIXM_20221013.0000.00_12.0_global_b4e7f89ff6.tgz](https://cdnorg.d.miui.com/V13.0.2.0.SJOMIXM/merlin_global_images_V13.0.2.0.SJOMIXM_20221013.0000.00_12.0_global_b4e7f89ff6.tgz)
- MD5 checked after download: `B4E7F89FF6317A634A80D37B7EF71A53`

The previous Ubuntu Touch vendor version was Android 11, so it did not satisfy the Android 12 requirement. The firmware above booted to the MIUI setup screen. We did **not** complete MIUI setup or test calls, SMS, LTE, Wi-Fi calling, or the other hardware functions on stock. The official LineageOS guide recommends testing stock functionality before continuing; do that when possible. The version above is the version used in this session, not a claim that it is still the newest Android 12 firmware.

### LineageOS

- ZIP: `lineage-24.0-20261020-nightly-merlinx-signed.zip`
- SHA-256: `5510B932FA80819BD22A8C955383E4CE9AAB2A38BD4CAC1094EDE7F7C2D13CF0`
- Lineage Recovery image was extracted from that same ZIP as `recovery.img`
- Extracted recovery SHA-256: `C001ECCF33D7CC8E3B535BF42427A53EBB02D7A967BCC0327C05049DFBCA97CB`

Use the current official downloads linked by the [LineageOS merlinx page](https://download.lineageos.org/devices/merlinx) when installing later. Do not assume the dated nightly above is still current.

## Computer setup used

- Windows 11 PC
- Android Platform Tools for Windows (`fastboot.exe` and `adb.exe`)
- USB cable and a USB-A port
- USB driver bound to the fastboot device as **WinUSB** (Windows hardware ID observed: `USB\VID_0BB4&PID_0C01`)

Use a reliable cable and preferably a USB-A 2.0 port or hub if fastboot hangs or loses the device. The phone should show **FASTBOOT** while running fastboot commands.

## Procedure

Read the current official LineageOS page from top to bottom first. The outline below records what worked for this phone and build.

### 1. Back up and check the bootloader

1. Back up phone data. This process wipes it.
2. Confirm the bootloader is unlocked. In fastboot mode:
   ```powershell
   fastboot devices
   fastboot getvar product
   fastboot getvar unlocked
   fastboot getvar rollback_ver
   ```
3. For this device, the observed values were `product: merlin`, `unlocked: yes`, and `rollback_ver: 2`. Stop if the product is unexpected, the bootloader is locked, or a firmware package's anti-rollback checks fail.
4. Match the firmware region to the device. The firmware used here was Global (`RJOMIXM` / `SJOMIXM`).

### 2. Return to Android 12 stock firmware

1. Download the Android 12 fastboot archive listed above and verify its MD5 before extracting.
2. Extract the archive. Its image files are under the extracted folder's `images` directory.
3. The Xiaomi archive includes `flash_all.bat`, `flash_all_except_data_storage.bat`, and `flash_all_lock.bat`. **Never use `flash_all_lock.bat` for this LineageOS installation**; keep the bootloader unlocked.
4. In this session, we used a PowerShell runner that checked the phone's product and rollback version, then flashed these partitions in order from the matching archive: `preloader`, `logo`, `tee1`, `tee2`, `scp1`, `scp2`, `sspm_1`, `sspm_2`, `lk`, `lk2`, `super`, `cache`, `recovery`, `boot`, `dtbo`, `vbmeta`, `spmfw`, `md1img`, `vbmeta_system`, `vbmeta_vendor`, `cust`, `exaid`, and `userdata`. Each `fastboot flash` completed successfully. The supplied Windows batch file was not used.
5. The optional `fastboot oem cdms` command returned `unknown command` on this phone. The following `fastboot reboot` succeeded; the optional command was not needed.
6. Let MIUI boot. The phone reached MIUI setup, confirming that Android 12 started.

**Important:** The LineageOS device guide also asks users to boot stock once and check functionality (including calls and SMS) before installing. This session continued after confirming the MIUI setup screen, without completing those tests.

### 3. Flash and boot Lineage Recovery

1. After stock boots, power the phone off.
2. Hold **Volume Down + Power** until FASTBOOT appears; connect the USB cable.
3. Extract `recovery.img` from the official LineageOS ZIP used for this installation, or download the current official recovery from the device page.
4. Verify the computer sees the device, then flash recovery:
   ```powershell
   fastboot devices
   fastboot flash recovery recovery.img
   ```
5. Do not boot MIUI after flashing recovery. Hold **Volume Up + Power** until the Redmi logo appears, then release. Verify that the **LineageOS Recovery** menu appears.

The recovery flash completed with `OKAY` for both sending and writing.

### 4. Format data and sideload LineageOS

1. In LineageOS Recovery, select **Factory reset → Format data / factory reset** and confirm. This erases internal storage and removes encryption.
2. Return to the main menu. Select **Apply update → Apply from ADB**.
3. On the computer, sideload the official LineageOS ZIP:
   ```powershell
   adb devices
   adb -d sideload lineage-24.0-20261020-nightly-merlinx-signed.zip
   ```
4. Wait for recovery to finish verification and installation. The computer reported `Total xfer: 1.00x`, and recovery reported installation success.
5. No Google Apps package was installed. If you want add-ons such as Google Apps, the LineageOS guide requires installing them from recovery before the first boot. Check that guide for the correct architecture and current package.
6. Select **Reboot system now**. The first boot reached the LineageOS welcome screen.

## Windows USB troubleshooting observed

- An empty `fastboot devices` result can be caused by the Windows USB driver or USB connection. First check Device Manager and try a different cable or USB-A port.
- The device appeared as `USB\VID_0BB4&PID_0C01` with a WinUSB driver. After rebooting from stock, Windows temporarily showed Code 10 and fastboot listed no device. Unplugging the phone, waiting a few seconds, and reconnecting to the same port cleared Code 10 and restored fastboot detection.
- An attempted Google USB Driver INF was not compatible with the device in this setup. An older Xiaomi driver download produced a catalog hash error, and Mi Unlock's driver button failed because its driver executable was missing. Those attempts were unnecessary once fastboot worked with WinUSB.
- If fastboot still shows no device, check the driver status and try another USB port before flashing anything. Do not continue when a flash command fails.

## References

- [Official LineageOS installation guide for Xiaomi Redmi Note 9 (merlinx)](https://wiki.lineageos.org/devices/merlinx/install/)
- [Official LineageOS downloads for merlinx](https://download.lineageos.org/devices/merlinx)
- [Xiaomi Global Android 12 fastboot firmware package used above](https://cdnorg.d.miui.com/V13.0.2.0.SJOMIXM/merlin_global_images_V13.0.2.0.SJOMIXM_20221013.0000.00_12.0_global_b4e7f89ff6.tgz)
- [Firmware listing with the published MD5](https://xmfirmwareupdater.com/miui/merlin/stable/V13.0.2.0.SJOMIXM/)

---

This is a personal installation record. LineageOS and Xiaomi firmware have their own licenses and support channels; this repository does not redistribute either firmware image.
