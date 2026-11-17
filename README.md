# PixelOS for Samsung Galaxy Note 20 Ultra (c2s)

Unofficial PixelOS (Android 17) port for the **Samsung Galaxy Note 20 Ultra, Exynos (SM-N986B, codename `c2s`)**, based on the LineageOS 24 device tree.

> **Status: work in progress.** The tree syncs, converts and passes Soong's dependency checks, but a full build has not completed yet (see *Hardware requirements*).

## Repositories

| Repository | Purpose |
|---|---|
| `pixelos-c2s-manifest` | This repo: the local manifest that ties everything together |
| `android_device_samsung_c2s` (branch `pixelos-port`) | Device tree converted from LineageOS to PixelOS |
| `android_hardware_samsung` (branch `pixelos-port`) | Samsung hardware repo with the lineage-sdk dependency removed from AdvancedDisplay and SamsungDoze |

All other projects (common tree, kernel, vendor blobs, Samsung SLSI HALs) come from **LineageOS** (`lineage-24.0`) and **TheMuppets**.

## Hardware requirements

Google's official AOSP requirements ([source.android.com](https://source.android.com/docs/setup/start/requirements)):

| | Minimum | Notes |
|---|---|---|
| **CPU** | 64-bit x86 | More cores = faster compiling |
| **RAM** | **64 GB** | Google builds Android on 72-core machines with 64 GB RAM |
| **Disk** | **400 GB free** | 250 GB for the source + 150 GB for the build, SSD strongly recommended |
| **OS** | 64-bit Linux | WSL2 works if the hardware above is met |

**From my own testing:** Soong's analysis step on Android 17 went past **55 GB of memory**. My 16 GB PC couldn't finish it, even with zram, a second zram device and swap on an SSD. So honestly, **64 GB of RAM is the real minimum** for this one.

## Before you build

All repos for this port are **public**, so anyone can sync and build it. A few things though:

1. **Check the hardware requirements above** (64 GB+ RAM, 400 GB+ free disk, 64-bit Linux). Anything weaker will most likely fail at the analysis step.
2. **Please message me before releasing any builds publicly.** You can reach me on Telegram through the **Contacts** section on [my website](https://blazepixel76.github.io/BlazeWebsite/). I'd like to test builds on my own device first, and keep credits right.
3. Found a bug or have a fix? Let me know there as well.

## Clone

```bash
mkdir pixelos && cd pixelos
repo init -u https://github.com/PixelOS-AOSP/android_manifest.git -b seventeen --git-lfs
git clone https://github.com/BlazePixel76/pixelos-c2s-manifest /tmp/c2s-manifest
mkdir -p .repo/local_manifests
cp /tmp/c2s-manifest/c2s.xml .repo/local_manifests/
repo sync -c -j8 --force-sync --no-clone-bundle --no-tags
```

## Patches

Some fixes live outside the device repos, so they're included here as patch files in [`patches/`](patches/). Apply them after `repo sync` and before building:

```bash
# Required: boot animation resolution fix
# (the N20U's display HAL can't remember a boot resolution, so without this the boot
#  animation glitches into a corner when you're using 1080p)
git -C frameworks/base   am ../pixelos-c2s-manifest/patches/0001-display-Persist-preferred-resolution-for-HWCs-withou.patch
git -C frameworks/native am ../pixelos-c2s-manifest/patches/0001-surfaceflinger-Restore-preferred-resolution-at-boot-.patch

# Optional: only for low-RAM build machines (around 16 GB)
# Makes Soong's analysis step use much less memory, at the cost of extra CPU time.
# Not needed if you have 64 GB+ RAM.
git -C build/soong am ../pixelos-c2s-manifest/patches/0001-LOCAL-Lower-soong_build-GC-target-for-16-GB-machines.patch
git -C build/soong am ../pixelos-c2s-manifest/patches/0002-LOCAL-Raise-soong_build-soft-memory-limit-to-32-GiB.patch
```

The paths above assume you cloned this repo next to your source folder. Adjust them if yours is somewhere else.

## Build

```bash
source build/envsetup.sh
breakfast c2s
m pixelos -j$(nproc)
```

The flashable zip is written to `out/target/product/c2s/PixelOS_*.zip`.

**Low-RAM machines:** lower `-j` (for example `-j6`), and use zram plus swap on an SSD. Never put swap on an HDD.

## Flashing

This device is usually flashed from **recovery** (for example Lineage Recovery):

1. Copy the zip to the SD card.
2. Recovery → **Apply update** → choose the zip.
3. Format data when coming from a different ROM.

Back up your **EFS** (`efs`, `sec_efs`) before flashing anything. It holds the IMEI.

## Changes from LineageOS

- `lineage_c2s.mk` → `custom_c2s.mk`, inheriting `vendor/custom/config/common_full_phone.mk`
- PixelOS flags: `TARGET_FACE_UNLOCK_SUPPORTED`, `TARGET_CALL_RECORDING_SUPPORTED`
- `hardware/lineage/generic-ims` added (required by the device tree, missing from the PixelOS manifest)
- `hardware/samsung`: AdvancedDisplay uses a local `FileUtils`, and SamsungDoze uses a local `ScreenType`, so neither needs lineage-sdk
- AdvancedDisplay shipped on c2s: Samsung mDNIe screen colour modes in Settings → Display
- PixelOS flags (face unlock, call recording, 1440p boot animation) set before the vendor config is inherited

## Credits

- [LineageOS](https://github.com/LineageOS): device, common, kernel and hardware trees
- [PixelOS](https://pixelos.net): the ROM
- [TheMuppets](https://github.com/TheMuppets): proprietary vendor blobs

## License

**Copyright © 2026 BlazePixel76**

This repository is licensed under the [Apache License 2.0](LICENSE).

The device tree, hardware code, kernel and other projects pulled in by this manifest belong to their respective authors (mainly [LineageOS](https://github.com/LineageOS), [PixelOS](https://pixelos.net) and [TheMuppets](https://github.com/TheMuppets)) and are covered by their own licenses.

## Reminder
Please Do Not Try To Build This If You Have A Potato PC Or Lower Than 64 GB Ram Unless You Want Your PC To Suffer All Day But then You Have to Meet Hardware Requirments As Said In AOSP website
