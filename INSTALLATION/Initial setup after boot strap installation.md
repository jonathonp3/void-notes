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

8. Enable Ethernet if you did not during installation int he chroot:
```bash
sudo ln -s /etc/sv/dhcpcd /var/service/
```

9. Setup wifi if you have not done already:

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

11. identify ip address
```bash
ip a | grep inet
```

12. Create ssh key If you want to continue the setup from a different computer:
```bash
ssh-keygen -t rsa
```

Login to void from another computer:
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

13. Install vim
Update xbps:
```bash
xbps-install -u xbps
```

14. List Available Locales:
```bash
locale -a
```

15. Set System Locale to Australian English:
```bash
sudo vim /etc/locale.conf
```
add
```bash
LANG=en_AU.UTF-8
```

16.  Generate the Locale (for glibc):
```bash
sudo xbps-reconfigure -f glibc-locales
sudo locale-gen
```

17. Update system:
```bash
sudo xbps-install -Syu
```

18. Restart
```bash
sudo reboot
```

