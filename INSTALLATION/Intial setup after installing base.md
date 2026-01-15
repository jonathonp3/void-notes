# Initial setup after boot strap installation

1. Login in to the root account:
```bash
root
```

3. Create user with admin privileges and set password:
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

4. Configure sudoers to allow admin privileges for new username
```bash
visudo
```

Uncomment or add the line (use the del key in Normal mode):
```bash
%wheel ALL=(ALL) ALL
```

5. logout:
```bash
logout
```

6. Login username:
```bash
username
```

7. identify ip address
```bash
ip a | grep inet
```

8. Create ssh key If you want to continue the setup from a different computer:
```bash
ssh-keygen -t rsa
```

Login to void from another computer:
```bash
ssh-keygen -t rsa
ssh-copy-id username@192.168.45.121
ssh username@192.168.45.121
```

9. Enable Ethernet if did not do it in the chroot:
```bash
sudo ln -s /etc/sv/dhcpcd /var/service/
```

10. Setup wifi if you have not done already:s

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

11. Enable wpa_supplicant service:
```bash
sudo ln -s /etc/sv/wpa_supplicant /var/service/
```

Check networking status:
```bash
sudo sv status wpa_supplicant
sudo sv status dhcpcd
```

12. Update hostname:
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

14. Update system:
```bash
sudo xbps-install -Syu
```

