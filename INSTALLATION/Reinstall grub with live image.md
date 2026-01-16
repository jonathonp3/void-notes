# Reinstall grub with void live xfce

1. Download the latest xfce live image at https://voidlinux.org/download/

2. Identify your SSD device (Example sdc). Make sure you have identified the correct device:
```bash
lsblk -f
```
4. Unmount usb-ssd:
```bash
sudo umount /dev/sdc1
```

5. Write the image to your flash drive (replace X with your device number) 
```bash
sudo dd bs=4M if=void-live-x86_64-20250202-xfce.iso of=/dev/sdX status=progress
```

6. Boot usb flash drive by entering the BIOS/UEFI by pressing a specific key during startup:
Common keys:
```bash
Del
F2
F10
F12
Esc
```
F12 works for gigabyte motherboards which are the only boards i use for Linux. Gaming boards have good Linux support.


7. Open a terminal and type to identify the EFI System Partition and the root partition which in my case is labelled void_data
```bash
lsblk -f
```

8. Mount Root partition
```bash
sudo mount -o subvol=@,compress=zstd /dev/nvme0n1p2 /mnt
```
9. Mount EFI Partition
```bash
sudo mount -o umask=0077 /dev/nvme0n1p1 /mnt/boot/efi
```

10. Prepare chroot environment:
```bash
sudo mount --bind /dev /mnt/dev
sudo mount --bind /proc /mnt/proc
sudo mount --rbind /sys /mnt/sys
sudo mount --bind /run /mnt/run
sudo cp /etc/resolv.conf /mnt/etc/resolv.conf
```

11. Chroot to /mnt
```bash
sudo chroot /mnt /bin/bash
```

12. Install GRUB:
```bash
xbps-install -S grub-x86_64-efi efibootmgr
```

13. Install grub:
```boot
grub-install --target=x86_64-efi --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```

14. Exit chroot
```bash
exit
```

15. Unmount /mnt
```bash
sudo umount -R /mnt
```

16. Shutdown
```bash
sudo poweroff
```



