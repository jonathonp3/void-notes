# Login to voidlinux network or live installer via ssh:

1. Login
```bash
login: root
```
passwd: voidlinux 

2. Add password authentication:
```bash
vi /etc/ssh/sshd_config
```

add

```bash
PermitRootLogin yes 
PasswordAuthentication yes 
```

2 Restart secure shell:
```bash
sv restart sshd
```

4. Create ssd key:
```bash
ssh-keygen -t rsa
```

6. Get ip address:
```bash
ip a | grep inet
```

9. Login to void from another computer. Update your ip address:

Example:
```bash
ssh-keygen -t rsa
ssh-copy-id username@192.168.45.121
ssh username@192.168.45.121
```
