# Reviving a 2012 Kindle Paperwhite 1: Jailbreak + KOReader + WeRead

[中文](README.md) | **English**

A first-generation Kindle Paperwhite from 2012, deep-discharged and shelved for years. It now runs the latest KOReader, reads WeRead (微信读书) online, and reads my own epubs offline.

This repo documents the full procedure and every pitfall. Documentation only: no firmware or jailbreak binaries are included, and every download points to the official source.

## End result

- KOReader v2026.07.1 (current release, fully usable on PW1)
- WeRead plugin: QR-code login, progress sync, chapter caching
- Local epub/mobi reading with typography far beyond the stock reader
- USBNetwork installed (optional, shell access)
- Firmware parked at 5.6.1.1 (the final PW1 release, so no OTA risk)

## Prerequisites

| Item | Note |
|---|---|
| Device | Kindle Paperwhite 1 (2012), 256 MB RAM |
| Firmware | 5.6.1.1 (highest version ever shipped for PW1) |
| Battery | Charge above 50% before flashing |
| Cable | Must be a data cable, not charge-only |

If the device is deep-discharged, charge from a wall adapter (an old 5V/1A brick is ideal) for 4+ hours, then hold the power button for 40 seconds to hard-reset.

## Key facts (read first, save hours)

1. **5.6.1.1 cannot be jailbroken directly.** The PW1 exploit only exists on 5.0–5.4.4.2.
2. **Downgrading is a means, not a destination.** Plant the jailbreak on old firmware, then go back up to 5.6.1.1 and apply the hotfix.
3. **Do not install KUAL/MRPI on the old firmware.** This was the biggest trap. The modern toolchain (2024+ KUAL Booklet, MRPI, USBNetwork packages) is built for the "new firmware + hotfix" environment. On 5.3.3 you will hit:
   - KUAL azw2 fails with "not signed by an authorized developer"
   - Update Your Kindle greyed out for every community package
   - Installers dropped in the root folder silently deleted at boot
   - The `;log mrpi` search-bar command does nothing
4. The jailbreak itself (a filename-injection exploit) is not subject to signature checks and can be re-run at any time.
5. 5.6.1.1 is the final PW1 firmware. Once you are back on it, Amazon has nothing newer to push.

## Full procedure

### 0. Back up

Mount over USB and copy the whole `documents/` folder to your computer.

### 1. Downgrade 5.6.1.1 → 5.3.3 (dirty-unplug method)

5.6.1.1 refuses a normal downgrade; you have to force it:

1. Enable Airplane Mode on the device
2. Connect over USB and place the official `update_kindle_5.3.3.bin` in the root folder
3. Wait 2 minutes so the system notices the file
4. **Keep USB connected**, hold the power button 15–20 seconds to force a restart
5. The device boots into the installer; do not unplug until the progress bar finishes

Firmware download (Amazon's official S3):
```
https://s3.amazonaws.com/G7G_FirmwareUpdates_WebDownloads/update_kindle_5.3.3.bin
```

### 2. Install the jailbreak

Use NiLuJe's K5 jailbreak (for 5.0–5.4.4.2):

1. Download the `kindle-jailbreak-1.16.N` package from the [Snapshots thread](https://www.mobileread.com/forums/showthread.php?t=225030)
2. Extract the inner `kindle-5.4-jailbreak.zip` and unzip its contents into the Kindle root folder
3. On the device: Settings → Menu → Update Your Kindle
4. `**** JAILBREAK ****` on screen means success

### 3. Upgrade back to 5.6.1.1

An up-to-date jailbreak survives the official upgrade.

1. Place the official `update_kindle_5.6.1.1.bin` in the root folder
2. Settings → Menu → Update Your Kindle
3. About 10 minutes; confirm firmware 5.6.1.1 afterwards

```
https://s3.amazonaws.com/G7G_FirmwareUpdates_WebDownloads/update_kindle_5.6.1.1.bin
```

### 4. Apply the jailbreak hotfix

1. Place `Update_jailbreak_hotfix_1.16.N_install.bin` (from the jailbreak package) in the root folder
2. Settings → Menu → Update Your Kindle

The hotfix exists precisely for the "jailbroken on old firmware, then upgraded to 5.6.x" scenario. After it, the jailbreak is stable on the new firmware.

### 5. Install KUAL + MRPI + USBNetwork

1. Download the KUAL, MRPI (kual-mrinstaller) and USBNetwork packages from the Snapshots thread
2. Copy MRPI's `extensions/` folder to the Kindle root
3. Create `mrpackages/` in the root and drop in:
   - `Update_KUALBooklet_v2.7.37_install.bin`
   - `Update_usbnet_0.22.N_install_touch_pw.bin` (PW1 uses the touch_pw variant)
4. On the home screen, type `;log mrpi` into the search bar and press enter
5. The MRPI installer screen appears and installs everything in the queue

### 6. Install KOReader + the WeRead plugin

1. From [KOReader releases](https://github.com/koreader/koreader/releases) download `koreader-kindle-v*.zip` (PW1 uses the plain kindle build, not legacy, pw2 or hf). Merge `koreader/` and `extensions/` into the root folder
2. From [weread.koplugin releases](https://github.com/finlater/weread.koplugin/releases) download the plugin zip and extract `weread.koplugin/` into `koreader/plugins/`
3. KUAL → Start KOReader (use the standard launcher, which stops the Amazon framework to free RAM; necessary on a 256 MB device)

### 7. WeRead login

1. In the WeRead mobile app: Me → Settings → WeRead Skill → enable and get an API Key
2. In KOReader: Tools → WeRead → Login via WeChat QR code
3. Scan with WeChat on your phone and confirm (if a 4-digit code appears on the phone, enter it in KOReader)

## Pitfalls

| Symptom | Cause | Fix |
|---|---|---|
| Plugged in but only shows "battery needs charging", won't boot | Deep-discharged Li-ion, protection circuit cut off | Wall-charge 4h+, hold power 40s |
| KUAL azw2 signature error on old firmware | Kindlet cert chain needs MKK; modern packages no longer target old firmware | Don't fix it. Upgrade to 5.6.1.1 and use the hotfix route |
| Update Your Kindle greyed out for community packages | Updater pre-check rejects them; old firmware lacks the modern jailbreak key environment | Same as above |
| Installers in root vanish after reboot | Boot-time signature check fails and deletes them silently | Same as above |
| `;log mrpi` does nothing | Search-bar hook not installed on old firmware | Works once on 5.6.1.1 |
| macOS refuses to eject the Kindle volume | Spotlight indexing holds it | `diskutil unmount force`, or sync then force-eject |
| KOReader file browser shows 0 files in a folder | Unsupported formats hidden by default; KOReader does not support azw3 (mobi works if DRM-free) | Convert azw3 to epub with Calibre; DRM'd Amazon purchases cannot be converted |
| Epubs dragged out of Apple Books won't open on Kindle | Books exports an unpacked folder (with iTunesMetadata.plist), not a single file | Re-zip to spec: `mimetype` first and stored uncompressed, drop the iTunes metadata |

## WeRead's epub cache

The plugin downloads chapters and assembles a complete epub in KOReader's cache folder. Finish a book and you have its epub on the device. **Personal use only; do not redistribute.** Redistribution is copyright infringement, and bulk scraping can trigger account bans.

## Credits

- [NiLuJe](https://www.mobileread.com/forums/member.php?u=27057) — maintainer of the K5 jailbreak, KUAL, MRPI and USBNetwork
- [koreader/koreader](https://github.com/koreader/koreader) — the reader itself
- [finlater/weread.koplugin](https://github.com/finlater/weread.koplugin) — the WeRead plugin
- [MobileRead forums](https://www.mobileread.com/forums/) — everyone who hit these walls first
- [oakreef's PW1 jailbreak write-up](https://oakreef.ie/bog/kindle-jailbreaking) — the article that pointed to the "go back up and hotfix" route

## Disclaimer

Flashing carries risk. This procedure was fully verified on one PW1, but you do it at your own risk. Power loss, unplugging mid-flash, or flashing the wrong file can leave the device unbootable.
