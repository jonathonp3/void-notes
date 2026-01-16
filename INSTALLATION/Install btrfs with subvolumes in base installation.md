# Void Installation Section: Partitioning with GParted
Preparation

    Recommended Installation Media: Void Linux Live XFCE ISO
    Preferred Partitioning Tool: GParted

Installation Steps
1. Boot into Live XFCE Environment

Use void-live-x86_64-xxxxxx-xfce ISO

2. Install GParted
```bash
sudo xbps-install -Sy
sudo xbps-install -u xbps
sudo xbps-install -S gparted
```

3. Partition Requirements

Partition Scheme: EFI


EFI System Partition
Creation Guidelines
```bash
Filesystem: FAT32
Size: 600 MB
Bootable Flag
```
Right-click on new EFI partition
Select 'Manage Flags'
Tick 'Boot'

Root Partition:	
```bash
Use btrfs file system 
Label 'void_data'
Swap Partition	4-12 GB	Linux Swap	
```

Caution Notes

Do not reformat existing EFI partition if you have another operating system installed.
Backup important data before partitioning

# Void base installation
 
4. To reformat the the parition if required for what ever reason in case you're are starting over: 
Identify partiton number
```bash
lsblk
```
Example to reformat:
```bash
sudo mkfs.btrfs -f /dev/nvme0n1p2
```
 
To reformat EFI Sysyem Partition (EFI)
(Caution if you have windows installed or another Linux distribution installed don't do this.)

Example (replace nvme0n1p1 with the specific device and partition):
```bash
sudo mkfs.vfat -F 32 /dev/nvme0n1p1
```
To reformat the swap partiton:
```bash
mkswap /dev/nvme0n1p3
```

5. Mount the Btrfs partition:
```bash
sudo mount /dev/nvme0n1p2 /mnt
```

6. Create subvolumes if not already created:
```bash
sudo btrfs subvolume create /mnt/@
sudo btrfs subvolume create /mnt/@home
sudo btrfs subvolume create /mnt/@snapshots
```

7. Umount after creating subvolumes:
```bash
sudo umount /mnt
```

8. Mount subvolumes for installation
Mount subvolumes and other partitions to /mnt and subdirectories:

```bash
sudo mount -o subvol=@,compress=zstd /dev/nvme0n1p2 /mnt
sudo mkdir -p /mnt/home /mnt/.snapshots
sudo mount -o subvol=@home,compress=zstd /dev/nvme0n1p2 /mnt/home
sudo mount -o subvol=@snapshots,compress=zstd /dev/nvme0n1p2 /mnt/.snapshots
```

9. Create and mount /boot/efi partition:
```bash
sudo mkdir -p /mnt/boot/efi
sudo mount -o umask=0077 /dev/nvme0n1p1 /mnt/boot/efi
```

Check partitions and mount points:
```bash
lsblk -f
```

10. Bootstrap the base system in /mnt:
```bash
sudo xbps-install -Sy -R https://repo-default.voidlinux.org/current -r /mnt base-system
```

11. Prepare chroot environment:
```bash
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --rbind /sys /mnt/sys
sudo mount --bind /run /mnt/run
sudo cp /etc/resolv.conf /mnt/etc/resolv.conf
```

12. Chroot to /mnt:
```bash
sudo chroot /mnt /bin/bash
```

13. Update fstab:
Copy the new UUID (check with lsblk -f)  to fstab

The root partition UUID in my example is 1c58e652-93e7-4dda-9620-efa45771c186 the new EFI System Partition is UUID=979D-3E72 after formatting.
```bash
vi /etc/fstab
```
Example:
```.bash
UUID=1c58e652-93e7-4dda-9620-efa45771c186  /               btrfs   subvol=@,defaults,compress=zstd             0  0
UUID=1c58e652-93e7-4dda-9620-efa45771c186  /home           btrfs   subvol=@home,defaults,compress=zstd         0  0
UUID=1c58e652-93e7-4dda-9620-efa45771c186  /.snapshots     btrfs   subvol=@snapshots,defaults,compress=zstd    0  0
UUID=979D-3E72                             /boot/efi       vfat    umask=0077                                  0  2
UUID=2ecbe876-0ae9-4ae0-bf51-aeb643ac7c94  none            swap    sw                                          0  0
tmpfs                                      /tmp            tmpfs   defaults,nosuid,nodev                       0  0
```
Press the Esc key :wq to write file and exit vi

14. Install GRUB:
```bash
xbps-install -S grub-x86_64-efi efibootmgr
```

15. Install grub:
```boot
grub-install --target=x86_64-efi --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```

16.  Enable DHCP for ethernet connection:
```bash
sudo ln -s /etc/sv/dhcpcd /var/service/
```

17. Setup wifi internet access for base installation (skip if not required)
```bash
xbps-install -S wpa_supplicant
```

Setup SSID and password:
```bash
sudo wpa_passphrase "SSID_NAME" "WiFi_PASSWORD" | sudo tee /etc/wpa_supplicant/wpa_supplicant.conf
```

Typical Generated File:
```bash
sudo wpa_passphrase "blue" "holekeeeptaste" | sudo tee /etc/wpa_supplicant/wpa_supplicant.conf
network={
	ssid="blue"
	#psk="holekeeeptaste"
	psk=c1f9c1d1e79b2c4e785e347430e99089fbdc0116269c64f2251ab8a1295284fb
}
```
Remove commented password line:
```bash
sudo sed -i '/^[[:space:]]*#psk=/d' /etc/wpa_supplicant/wpa_supplicant.conf
```

or manually remove the password:

```bash
sudo vi /etc/wpa_supplicant/wpa_supplicant.conf
```
Remove the password as it no longer required
```bash
#psk="holekeeeptaste"
```

18. Enable wpa_supplicant service:
```bash
sudo ln -s /etc/sv/wpa_supplicant /var/service/
```

19. Unmount /mnt
```bash
sudo umount -R /mnt
sudo swapoff /dev/nvme0n1p5
```

20. Shutdown and read 'Intial setup after installing base.md'
```bash
sudo poweroff
```





