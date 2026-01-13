# Void Linux SSD btrfs Migration Guide

This guide walks you through migrating a Void Linux clean freshly imaged **(unused)** root partition from ext4 to btrfs on an SSD, which includes backing up original image data, reformatting the partition, restoring image data, and updating the boot configuration.

---

## Download Void Linux ARM Image for Raspberry Pi 5

1. Download the ARM image:
```bash
wget https://repo-default.voidlinux.org/live/current/void-rpi-aarch64-20250202.img.xz
```

2. Decompress the image:
```bash
xz -d void-rpi-aarch64-20250202.img.xz
```

3. Identify your SSD device (Example sdc). Make sure you have identified the correct device:
```bash
lsblk -f
```
4. Unmount usb-ssd:
```bash
sudo umount /dev/sdc1
```

5. Write the image to your SSD (replace X with your device number) Example /dev/sdc:
```bash
sudo dd bs=4M if=void-rpi-aarch64-20250202.img of=/dev/sdX status=progress
```

**Step-by-Step Guide**

Current Setup

Leave the usb ssd in 
You have imaged the SSD (/dev/sdX) with dd, resulting in:

    sdc1 (FAT16) — boot partition
    sdc2 (ext4) — root/data partition

To Do:

    Mount and back up /dev/sdc2 data
    Reformat /dev/sdc2 with btrfs.
    Copy the backed-up data back to the btrfs partition.
    Update the filesystem table (fstab) and boot configuration.


1.  Mount Partition 2 and Back Up Data
    Create mount points
```bash
sudo mkdir -p /mnt/sdc2 /mnt/backup_sdc2
```
Mount the ext4 root partition
```bash
sudo mount /dev/sdc2 /mnt/sdc2
```
Back up data to a location (about 100mb for the unused image)
```bash
sudo rsync -aAXHv --no-xattrs /mnt/sdc2/ /mnt/backup_sdc2/
```

2. Unmount and Reformat Partition 2 as btrfs
```bash
sudo umount /mnt/sdc2
```

Check the full disk layout
```bash
lsblk
sudo parted /dev/sdc print
```
Example with parted:
```bash
sudo parted /dev/sdc
(parted) print
# Note the start sector of sdc2

# Delete partition 2
(parted) rm 2

# Create new partition 2 starting at the same sector but extending to the end
(parted) mkpart primary <start_sector> 100%
(parted) print
(parted) quit
```
Double-check the device before formatting!
```bash
lsblk -f
sudo mkfs.btrfs -f /dev/sdc2
```
Mount the new btrfs partition
```bash
sudo mount /dev/sdc2 /mnt/sdc2
```
Create subvolumes:
```bash
sudo btrfs subvolume create /mnt/sdc2/@
sudo btrfs subvolume create /mnt/sdc2/@home
sudo btrfs subvolume create /mnt/sdc2/@snapshots
```
Unmount the partition:
```bash
sudo umount /mnt/sdc2
```

3.  Mount Subvolumes and Restore Data
Mount the root subvolume @:
```bash
sudo mount -o subvol=@ /dev/sdc2 /mnt/sdc2
```

3. Restore all data except /home into root subvolume
Use rsync to copy data to subvolume
```bash
sudo rsync -aAXHv --no-xattrs /mnt/backup_sdc2/ /mnt/sdc2/
```

4. Update /etc/fstab
Find the new UUID of the btrfs partition:
```bash
sudo blkid /dev/sdc2
```
Edit the fstab on the restored root:
```bash
sudo vim /mnt/sdc2/etc/fstab
```
Update the root entry (/) to use btrfs and with the new UUID. Example:
```bash
UUID=4ef4d54d-7987-4199-818e-5a2c35bf8f8d /       btrfs defaults,compress=zstd,subvol=@      0 1
UUID=4ef4d54d-7987-4199-818e-5a2c35bf8f8d /home   btrfs defaults,compress=zstd,subvol=@home  0 2
```

5. Update Raspberry Pi Boot Configuration (cmdline.txt)
Mount the boot partition:
```bash
sudo mkdir -p /mnt/sdc1
sudo mount /dev/sdc1 /mnt/sdc1

# Find the PARTUUID with the output of:
```bash
sudo blkid -s PARTUUID -o value /dev/sdc2
```
Edit cmdline.txt to set the correct root partition by PARTUUID and add rootflags=subvol=@:
```bash
sudo vim /mnt/sdc1/cmdline.txt
```
<span style="color: purple;"For example:</b>
```bash
root=PARTUUID=5eb9c727-02 rootflags=subvol=@ rw rootwait console=ttyAMA0,115200 kgdboc=ttyAMA0,115200 console=tty1 smsc95xx.turbo_mode=N dwc_otg.lpm_enable=0 loglevel=4 elevator=noop
```

6. Unmount and test on Raspberry Pi
```bash
sudo umount /mnt/sdc2
sudo umount /mnt/sdc1
```

<b>Notes</b>

 - Always double-check the device name before formatting the Raspberry Pi to avoid formatting the incorrect device.

- Use --no-xattrs with rsync to avoid permission issues with extended attributes. Works without errors with a clean freshly unused image.

- For Raspberry Pi, ensure cmdline.txt is a single line with no line breaks.

- Validate UUID and PARTUUID consistency between /etc/fstab and cmdline.txt if does not correctly. Make sure you add *rootflags=subvol=@*

- Do not remove the empty home directory in the root partition. It needs to be there otherwise subvolume @home will not mount.







