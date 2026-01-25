# Start Syncthing to start at boot as a service 

Make sure rsyslog is installed (Skip this if you have done this already):

1. Install rsyslog
```bash
sudo xbps-install -S rsyslog
```
2. Start and enable ryslog at boot
```bash
sudo ln -s /etc/sv/rsyslogd /var/service
reboot
```

3. Install syncthing from repository:

```bash
sudo xbps-install -S syncthing
```

4. Create the Syncthing service directory:
```bash
sudo mkdir -p /etc/sv/syncthing
```

5. Start and enable syncthing to start at boot. Specify your home directory:
```bash
sudo vim /etc/sv/syncthing/run
```

add
```bash
#!/bin/sh
exec runuser -u jonathon -- syncthing serve --home=/home/jonathon/.local/state/syncthing --no-browser --no-restart
```

6. Make script executable:
```bash
sudo chmod +x /etc/sv/syncthing/run
```

7. Add log directory:
```bash
sudo mkdir /etc/sv/syncthing/log
```

8. Add log script
```bash
sudo vim /etc/sv/syncthing/log/run
```

add
```bash
#!/bin/sh
exec vlogger -t syncthing -p daemon.info
```

9. Make script executable:
```bash
sudo chmod +x /etc/sv/syncthing/log/run
```

10. Start and enable service
```bash
sudo ln -s /etc/sv/syncthing /var/service/
```

11. Manage service:
```bash
sudo sv restart syncthing
sudo sv stop syncthing
sudo sv start syncthing
sudo sv status syncthing
```

Disable service:
```bash
sudo rm /var/service/syncthing
```

12. View logs:
```bash
grep syncthing /var/log/messages
```

13. Optional: Separate Syncthing Log File:
```bash
sudo vim /etc/rsyslog.d/syncthing.conf
```

add
```bash
if $programname == 'syncthing' then /var/log/syncthing.log
& stop
```

Restart rsyslog:
```bash
sudo sv restart rsyslog
```

Check logs:
```bash
tail -f /var/log/syncthing.log
```




