# Kobo Init Script - Alternative Root Filesystem Boot (Buildroot)

## Overview

This init script allows booting an alternative root filesystem on a Kobo e-reader device by hijacking the boot process before the stock Kobo OS has time to fully initialize. It uses the stock system as a lightweight initramfs to mount and pivot to a custom rootfs image stored on the internal storage partition.

**Key concept**: By replacing the stock `/sbin/init` with this script, we can intercept the boot process, mount an alternative system image from `/mnt/onboard`, perform a `pivot_root`, and boot into a completely different Linux distribution while using the same kernel.

> **⚠️ Device Compatibility**: Currently only tested and working on the **Kobo Clara Colour**. Other Kobo models may require adjustments to device paths and trigger mechanisms.

## How It Works

### Boot Flow

1. **Early Boot Interception**: The script replaces the stock init, running immediately after kernel initialization
2. **Trigger Detection**: Waits up to 10 seconds for a physical button push event to trigger alternative boot
3. **Storage Mount**: Mounts the onboard storage partition (`/dev/mmcblk0p12`) where the alternative OS image resides
4. **Image Mount**: Mounts the ext4 rootfs image located at `.buildroot/buildroot-kobo.img`
5. **Pivot Root**: Performs `pivot_root` to switch from the stock rootfs to the new one
6. **Execute Init**: Launches `/sbin/init` from the alternative rootfs
7. **Fallback**: If any step fails or no trigger is detected, falls back to the stock Kobo init

### Technical Details

#### Device Configuration
- **Root Device**: `/dev/mmcblk0p10` - Stock Kobo root partition
- **Onboard Device**: `/dev/mmcblk0p12` - Userdata storage partition (where the image is stored)
- **Trigger Device**: `/dev/input/event1` - Touch screen event device
- **Image Path**: `.buildroot/buildroot-kobo.img` - Alternative rootfs image location on onboard storage, check [buildroot-kobo](https://github.com/Spin42/buildroot-kobo)

#### Boot Trigger
The script monitors `/dev/input/event1` for physical input:
- Waits up to 10 seconds after the device appears
- If a press is detected during this window, alternative boot is triggered
- If timeout occurs or no device is found, falls back to stock boot

## Use Cases

- **Testing custom Linux distributions** on Kobo hardware without modifying the stock system
- **Dual-boot setup** where the user can choose between stock Kobo OS and a custom system
- **Running alternative software stacks** (different init systems, desktop environments, etc.)

## Installation

1. Replace `/sbin/init` with this `init` script
2. Make it executable: `chmod +x /sbin/init`
3. Place your alternative rootfs image at `.buildroot/buildroot-kobo.img` on the Kobo's internal storage

## Limitations

- Uses the **same kernel** as the stock Kobo OS (kernel modules must be compatible)
- Alternative rootfs must be an ext4 filesystem image
- Requires modification of the stock system partition, which may be overwritten at next update

## Security Considerations

⚠️ **Warning**: Replacing the init script requires root access and modifies critical system files. Incorrect implementation can render the device unbootable. Use at your own risk. This is provided as is and without any warranty.
