# Install Gnome Desktop

Setup logging service

1. Install rsyslog
```bash
sudo xbps-install -S rsyslog
```
2. Start and enable ryslog at boot
```bash
sudo ln -s /etc/sv/rsyslogd /var/service
reboot
```

3. Install and Enable Core Service:
```bash
sudo xbps-install -S dbus
```
4. Start and enable dbus at boot:
```bash
sudo ln -s /etc/sv/dbus /var/service/
```

This tells us the dbus service is using the vlogger utility for logging:  
```bash
cat /etc/sv/dbus/log/run
```
Output:
```bash
#!/bin/sh
exec vlogger -t dbus -p daemon
```

With -t dbus -p daemon, it sends logs to the syslog daemon, tagged as dbus with the daemon facility. syslog is handled by rsyslog. See /var/log/messages

4. Setup automatic network discovery with logging: 
```bash
sudo xbps-install -S avahi avahi-utils
sudo ln -s /etc/sv/avahi-daemon /var/service/
```
Setup logging:
```bash
sudo mkdir /etc/sv/avahi-daemon/log
sudo vim /etc/sv/avahi-daemon/log/run
```

add
```bash 
#!/bin/sh
exec vlogger -t avahi-daemon -p daemon.info
```

Make script executable:
```bash
sudo chmod +x /etc/sv/avahi-daemon/log/run
```

Restart service (might need a reboot to view logs)
```bash
sudo sv restart avahi-daemon
```

To view all avahi-daemon log entries:
```bash
grep avahi-daemon /var/log/messages
```

Real-time Monitoring:
```bash
tail -f /var/log/messages | grep avahi-daemon
```


