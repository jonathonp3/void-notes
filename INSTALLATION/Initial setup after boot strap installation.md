# Initial setup after boot strap installation

1. Login in to the root account:
```bash
root
```

2. Create user with admin privileges and set password:
```bash
useradd -m -s /bin/bash username
```
Add wheel to group:
```bash
usermod -aG wheel username
```
Set user password:
```bash
passwd username
```

3. Configure sudoers to allow admin privileges for new username
```bash
visudo
```

Uncomment or add the line (use the del key in Normal mode):
```bash
%wheel ALL=(ALL) ALL
```

4. Add password authentication sshd:
```bash
sudo vim /etc/ssh/sshd_config
```

add

```bash
PermitRootLogin no
PasswordAuthentication yes 
```

5. Start and enable openssh:
```bash
sudo ln -s /etc/sv/sshd /var/service/
sudo sv status sshd
```

6. logout:
```bash
logout
```

7. Login username:
```bash
username
```

8. Enable Ethernet if you did not during installation in the chroot:
```bash
sudo ln -s /etc/sv/dhcpcd /var/service/
```

9. Setup wifi in case you did not do during chroot:

Setup ssid and password:
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
or 

Manually remove the commented out password entry:
```bash
sudo vim /etc/wpa_supplicant/wpa_supplicant.conf
```
Remove password:
```bash
#psk="holekeeeptaste"
```

10. Enable wpa_supplicant service:
```bash
sudo ln -s /etc/sv/wpa_supplicant /var/service/
```

Check networking status:
```bash
sudo sv status wpa_supplicant
sudo sv status dhcpcd
```

11. Identify your ip address:
```bash
ip a | grep inet
```

12. Create ssh key If you want to continue the setup from a different computer:
```bash
ssh-keygen -t rsa
```

Login to void from another computer. Update your ip address:
```bash
ssh-keygen -t rsa
ssh-copy-id username@192.168.45.121
ssh username@192.168.45.121
```

13. Update hostname:
```bash
sudo vim /etc/hostname
```
add computer name:
```bash
myhostname
```
press the esc key then type :wq to save and exit

14. Update xbps:
```bash
sudo xbps -Sy
xbps-install -u xbps
```
15. Install vim if you did do so during installation:
```bash
xbps-install -S vim
```

16. Set System Locale to Australian English:
```bash
sudo vim /etc/default/libc-locales
```
add

en_AU.UTF-8 UTF-8

17. Set system's default locale:
```bash
sudo vim /etc/locale.conf
```
add

```bash
LANG="en_AU.UTF-8"
LC_ALL="en_AU.UTF-8"
```

18.  Apply the locale
```bash
sudo reboot
```

19. Check Locale Settings:
```bash
locale
```

20. Set mirrors:
```bash
sudo vim /etc/xbps.d/00-repository-main.conf
```

add

```bash
# Prioritize Fastly CDN (Tier 1 - globally distributed, often fastest, always fresh)
repository=https://repo-fastly.voidlinux.org/current
repository=https://repo-fastly.voidlinux.org/current/nonfree
repository=https://repo-fastly.voidlinux.org/current/debug

# Australian Tier 2 mirrors (as fallbacks, choose one or both)
# AARNet - Canberra, Australia
repository=https://mirror.aarnet.edu.au/pub/voidlinux/current
repository=https://mirror.aarnet.edu.au/pub/voidlinux/current/nonfree
repository=https://mirror.aarnet.edu.au/pub/voidlinux/current/debug

# Swinburne University - Melbourne, Australia
repository=https://ftp.swin.edu.au/voidlinux/current
repository=https://ftp.swin.edu.au/voidlinux/current/nonfree
repository=https://ftp.swin.edu.au/voidlinux/current/debug

# Optional: Other reliable Tier 1 mirrors as very general fallbacks
# repository=https://repo-fi.voidlinux.org/current
# repository=https://repo-de.voidlinux.org/current
# repository=https://mirrors.servercentral.com/voidlinux/current
```

21. Check mirrors:
```bash
sudo xbps-install -S
```

22. Update system:
```bash
sudo xbps-install -Syu
```

23. Restart
```bash
sudo reboot
```

