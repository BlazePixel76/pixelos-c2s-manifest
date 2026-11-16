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

## Access (private repos)

These repos are **private**, so if you want to build this:

1. **Message me first** and I'll give you access. You can reach me on Telegram through the **Contacts** section on [my website](https://blazepixel76.github.io/BlazeWebsite/). `repo sync` won't work until I add you.
2. **Tell me your build specs** so we can confirm they meet the requirements above (64 GB+ RAM, 400 GB+ free disk, 64-bit Linux). Anything weaker will most likely fail at the analysis step, so it saves us both time.
3. Once you've got access, log in so git can pull the private repos:

```bash
gh auth login
gh auth setup-git
```

## Clone

```bash
mkdir pixelos && cd pixelos
repo init -u https://github.com/PixelOS-AOSP/android_manifest.git -b seventeen --git-lfs
git clone https://github.com/BlazePixel76/pixelos-c2s-manifest /tmp/c2s-manifest
mkdir -p .repo/local_manifests
cp /tmp/c2s-manifest/c2s.xml .repo/local_manifests/
repo sync -c -j8 --force-sync --no-clone-bundle --no-tags
```

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

## Credits

- [LineageOS](https://github.com/LineageOS): device, common, kernel and hardware trees
- [PixelOS](https://pixelos.net): the ROM
- [TheMuppets](https://github.com/TheMuppets): proprietary vendor blobs

## License

Copyright (C) 2026 BlazePixel76

This repository is licensed under the [Apache License 2.0](LICENSE).

The device tree, hardware code, kernel and other projects pulled in by this manifest belong to their respective authors (mainly [LineageOS](https://github.com/LineageOS), [PixelOS](https://pixelos.net) and [TheMuppets](https://github.com/TheMuppets)) and are covered by their own licenses.
