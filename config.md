# Config

Compiling a custom LEDE/OpenWrt firmware for the GL.iNET GL-MT1300 (Beryl) to support **External Storage Expansion (Extroot)** is a smart move. Since the MT1300 has limited internal flash, offloading the root filesystem to an SD card allows you to install heavy plugins like PassWall without worrying about space.

Here are the configuration notes for the essential modules you need to select in `make menuconfig` to enable SD card support and Extroot.

## 1. Required Kernel Modules (Drivers)

These modules are necessary for the system to recognize the SD card reader and the hardware interface.

-   **Kernel modules** ---> **Other modules**:
    -   `kmod-mmc`
    -   `kmod-sdhci`
    -   `kmod-sdhci-mt7621` (Specific driver for the MT1300 SDXC controller)
-   **Kernel modules** ---> **USB Support**:
    -   `kmod-usb-storage`
    -   `kmod-usb-storage-extras`
    -   `kmod-usb-ohci`
    -   `kmod-usb-uhci`
    -   `kmod-usb2`
    -   `kmod-usb3`

## 2. Filesystem Support

Since you will likely format your SD card to **ext4** (recommended for Linux systems), you need the following:

-   **Kernel modules** ---> **Filesystems**:
    -   `kmod-fs-ext4` (Crucial for the root partition)
    -   `kmod-fs-vfat` (Optional, if you want to read FAT32 partitions)
    -   `kmod-fs-ntfs3` (Optional)

## 3. Storage Management Utilities

These tools are required to handle partitioning and the actual "mounting" of the system to the SD card.

-   **Base system**:
    -   `block-mount` (The core tool for the `mount_root` and `fstab` functionality)
-   **Utilities** ---> **Disc**:
    -   `fdisk` (To partition the SD card)
    -   `blkid`
-   **Utilities** ---> **Filesystem**:
    -   `e2fsprogs` (Specifically `mkfs.ext4` to format the SD card)

## 4. LuCI Interface (Web UI)

To manage the mount points and storage via the browser, include these:

-   **LuCI** ---> **Applications**:
    -   `luci-app-mountd`
    -   `luci-app-fstab` (Highly recommended for visual configuration of Extroot)

## 5. Summary Checklist for `make menuconfig`

| **Category** | **Item to Select**  | **Purpose**                        |
| ------------ | ------------------- | ---------------------------------- |
| **Kernel**   | `kmod-sdhci-mt7621` | Hardware driver for SD slot        |
| **Kernel**   | `kmod-fs-ext4`      | Support for Linux filesystem       |
| **Base**     | `block-mount`       | Enables the Extroot mechanism      |
| **Utils**    | `e2fsprogs`         | Used to format the SD card to ext4 |
| **LuCI**     | `luci-app-fstab`    | UI for managing the expansion      |

## Post-Compilation Workflow

Once you flash this firmware, the general steps to perform the expansion are:

1.  **Partition**: Use `fdisk /dev/mmcblk0` to create a new partition on the SD card.
2.  **Format**: Use `mkfs.ext4 /dev/mmcblk0p1` to format it.
3.  **Transfer**: Copy the current `/overlay` data to the SD card.
4.  **Configure**: Edit `/etc/config/fstab` to mount the SD card partition as `/overlay`.
5.  **Reboot**: After rebooting, your "Free Space" in the software tab should reflect the size of your SD card.