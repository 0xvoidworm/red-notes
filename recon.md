# Reconnaissance

## Port Scanning

### Quick scan — top ports
```bash
nmap -sC -sV -oN scan.txt $IP
```

### Full TCP
```bash
nmap -p- --min-rate 5000 -oN full.txt $IP
```

### UDP top 20
```bash
sudo nmap -sU --top-ports 20 -oN udp.txt $IP
```

## Web Enumeration

### Directory brute-force
```bash
# gobuster
gobuster dir -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -o dirs.txt

# ffuf (faster, auto-calibrate wildcard)
ffuf -u http://$IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -ac -o ffuf.json -of json
```

### Virtual host enumeration
```bash
ffuf -u http://$IP -H "Host: FUZZ.$DOMAIN" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -ac
```

### Technology fingerprinting
```bash
whatweb http://$IP
curl -sI http://$IP
```

## DNS

### Zone transfer attempt
```bash
dig axfr @$IP $DOMAIN
```

### Subdomain enum
```bash
dig any $DOMAIN @$IP
dnsenum $DOMAIN
```

## SMB (445)

```bash
# null session
smbclient -N -L //$IP
crackmapexec smb $IP -u '' -p '' --shares

# with creds
smbclient //$IP/share -U 'user%pass'
crackmapexec smb $IP -u user -p pass --shares
```

## SNMP (161/udp)

```bash
snmpwalk -v2c -c public $IP
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt $IP
```

## LDAP (389)

```bash
ldapsearch -x -H ldap://$IP -b "dc=domain,dc=local"
```
