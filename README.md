# PixelOS for Galaxy Note 20 Ultra (c2s)

Unofficial PixelOS (Android 17) for the Galaxy Note 20 Ultra Exynos (SM-N986B, `c2s`), based on the LineageOS 24 device tree.

## Downloads

Latest build: [Releases](https://github.com/BlazePixel76/pixelos-c2s-manifest/releases/latest) (ROM zip is on [Gofile](https://gofile.io/d/RFSln38u), recovery is attached to the release)

Optional root kernel: [BakaKrnl](https://github.com/BlazePixel76/BakaKrnl-c2s)

## Installing

1. Back up your data and EFS first (`efs`, `sec_efs`, it has your IMEI).
2. Flash the PixelOS recovery from the release (Odin AP with the .tar, or the .img).
3. Reboot to recovery, format data (only needed when coming from another ROM).
4. Apply update, pick the ROM zip, reboot.

Builds are signed with my own keys, so you need my recovery for updates to install without a signature warning.

## Repos

| Repo | What it is |
|---|---|
| `pixelos-c2s-manifest` | This repo, the local manifest |
| [`android_device_samsung_c2s`](https://github.com/BlazePixel76/android_device_samsung_c2s) (`pixelos-port`) | Device tree, converted from LineageOS to PixelOS |
| [`android_hardware_samsung`](https://github.com/BlazePixel76/android_hardware_samsung) (`pixelos-port`) | Samsung hardware repo, AdvancedDisplay and SamsungDoze without lineage-sdk |

Everything else (common tree, kernel, SLSI HALs, vendor blobs) comes from LineageOS and TheMuppets (`lineage-24.0`).

## Building

You'll need a lot of RAM and disk space. Google recommends 64 GB RAM and 400 GB free disk. I built this on 16 GB RAM with the Soong patches below, zram and swap, and it worked, but analysis alone eats way more memory than you have. Also don't build on an HDD unless you're fine waiting days (mine took 3 to 4).

### Sync

```bash
mkdir pixelos && cd pixelos
repo init -u https://github.com/PixelOS-AOSP/android_manifest.git -b seventeen --git-lfs
git clone https://github.com/BlazePixel76/pixelos-c2s-manifest /tmp/c2s-manifest
mkdir -p .repo/local_manifests
cp /tmp/c2s-manifest/c2s.xml .repo/local_manifests/
repo sync -c -j8 --force-sync --no-clone-bundle --no-tags
```

If the build fails with "presigned apk" errors, the GMS files are still LFS pointers. Run `git lfs pull` inside `vendor/pixel/gms`.

### Patches

Apply these after syncing, from the root of the source folder:

```bash
PATCHES=/tmp/c2s-manifest/patches

# Required: fixes the boot animation glitching into a corner at 1080p
# (the display HAL doesn't remember the boot resolution)
git -C frameworks/base   am "$PATCHES"/0001-display-Persist-preferred-resolution-for-HWCs-withou.patch
git -C frameworks/native am "$PATCHES"/0001-surfaceflinger-Restore-preferred-resolution-at-boot-.patch

# Optional: only for ~16 GB RAM machines, makes Soong analysis use less memory
git -C build/soong am "$PATCHES"/0001-LOCAL-Lower-soong_build-GC-target-for-16-GB-machines.patch
git -C build/soong am "$PATCHES"/0002-LOCAL-Raise-soong_build-soft-memory-limit-to-32-GiB.patch
```

### Build

```bash
source build/envsetup.sh
breakfast c2s
m pixelos -j$(nproc)
```

The zip ends up in `out/target/product/c2s/`. On low-RAM machines use a lower `-j` (like `-j6`) and zram plus swap on an SSD.

## Changes from LineageOS

- `lineage_c2s.mk` renamed to `custom_c2s.mk`, inheriting `vendor/custom/config/common_full_phone.mk`
- PixelOS flags (face unlock, call recording, 1440p boot animation) set before the vendor config
- Added `hardware/lineage/generic-ims` (needed by the device tree, missing from the PixelOS manifest)
- `hardware/samsung`: local `FileUtils` for AdvancedDisplay and local `ScreenType` for SamsungDoze, so neither needs lineage-sdk
- AdvancedDisplay included, so mDNIe colour modes show up in Settings > Display

## Credits

- [LineageOS](https://github.com/LineageOS) for the device, common, kernel and hardware trees
- [PixelOS](https://pixelos.net) for the ROM
- [TheMuppets](https://github.com/TheMuppets) for the vendor blobs

## License

Copyright © 2026 BlazePixel76

This repo is under the [Apache License 2.0](LICENSE). Everything the manifest pulls in belongs to its authors (mainly LineageOS, PixelOS and TheMuppets) and has its own license.
