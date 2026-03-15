# Linux Privilege Escalation Notes

## Ressources

- https://github.com/MattiaCossu/PE-Linux?tab=readme-ov-file#orgb49dfc0
- https://vulp3cula.gitbook.io/hackers-grimoire/post-exploitation/privesc-linux
- https://gtfobins.github.io/
- https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#sudo-and-suid

---

# Enumeration de base

## Processus

Voir les processus, l'utilisateur qui les a lancés et s'ils ont un TTY.

```bash
ps axu
```

---

## Connexions réseau

Voir les connexions réseau :

```bash
ss -anp
```

Alternative :

```bash
netstat -anp
```

---

## Tâches Cron

Lister les tâches cron :

```bash
ls -lah /etc/cron*
```

Vérifier le PATH dans :

```bash
/etc/crontab
```

Objectif : voir si un **binaire peut être spoofé via l'ordre du PATH** (ordre gauche → droite).

Voir les logs cron :

```bash
grep "CRON" /var/log/cron.log
```

---

## Timers systemd

Similaire aux cron :

```bash
systemctl list-timers
```

---

## Applications installées

Sur systèmes Debian / Ubuntu :

```bash
dpkg -l
```

---

## Répertoires accessibles en écriture

```bash
find / -writable -type d 2>/dev/null
```

---

## Disques et montages

Voir les systèmes montés :

```bash
mount
```

Configuration des montages :

```bash
cat /etc/fstab
```

Lister les disques :

```bash
lsblk
```

---

## Kernel Modules

Modules chargés :

```bash
lsmod
```

Informations détaillées sur un module :

```bash
/sbin/modinfo libata
```

---

# SUID / Privilege Escalation

## Recherche des fichiers SUID

```bash
find / -perm -u=s -type f 2>/dev/null
```

Ensuite analyser les binaires via :

https://gtfobins.github.io/

---

## Scripts d'automatisation

Scripts qui automatisent l'énumération :

### unix-privesc-check

```bash
./unix-privesc-check standard > output.txt
```

### LinPEAS

Très bon script avec **indication couleur du niveau d'exploitation**.

---

# Exploits Kernel

Selon l'architecture cible.

Principe :

1. Identifier version kernel
2. Chercher exploit correspondant
3. Compiler sur la machine cible

```bash
gcc exploit.c -o exploit
```

Attention : gcc doit être présent.

Voir page **570 du PDF OWASP**.

---

# Pager Escapes

Les **pagers** permettent parfois d'exécuter des commandes.

Exemple dans **vi** :

```
!/bin/bash
```

Si exécuté dans un contexte `sudo`, cela peut donner un **shell root**.

Voir section **27.2.2 du PDF**.

---

# Commandes utiles

## Capture de mots de passe potentiels

```bash
sudo tcpdump -i lo -A | grep "pass"
```

`-A` : affichage ASCII.

---

## Monitoring de processus

```bash
watch -n 1 "ps -aux | grep pass"
```

---

# Liste usuelle des SUID

## Exemple Debian

```bash
find / -perm -u=s -type f 2>/dev/null
```

```
/usr/bin/find
/usr/bin/chsh
/usr/bin/fusermount
/usr/bin/chfn
/usr/bin/passwd_flag
/usr/bin/passwd
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/ntfs-3g
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/bwrap
/usr/bin/su
/usr/bin/umount
/usr/bin/mount
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/xorg/Xorg.wrap
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/sbin/pppd
```

---

# Capabilities

Lister les capabilities :

```bash
getcap -r / 2>/dev/null
```

## Exemple OSPC C / Charlie

```
/usr/bin/mtr-packet cap_net_raw=ep
/usr/bin/ping cap_net_raw=ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper cap_net_bind_service,cap_net_admin=ep
/snap/core20/1695/usr/bin/ping cap_net_raw=ep
/snap/core20/1634/usr/bin/ping cap_net_raw=ep
```

---

## Exemple Ubuntu 22.04

```
/usr/bin/ping cap_net_raw=ep
/usr/bin/mtr-packet cap_net_raw=ep
/usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper cap_net_bind_service,cap_net_admin=ep
/snap/core20/1695/usr/bin/ping cap_net_raw=ep
/snap/core20/1623/usr/bin/ping cap_net_raw=ep
```

---

# SUID Ubuntu 16.04

Kernel :

```
4.4.0-116-generic
```

Command :

```bash
find / -perm -u=s -type f 2>/dev/null
```

Résultat :

```
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/bin/sudo
/usr/bin/newuidmap
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/newgidmap
/usr/bin/chfn
/usr/bin/at
/usr/bin/pkexec
/bin/su
/bin/fusermount
/bin/ping
/bin/ntfs-3g
/bin/mount
/bin/umount
/bin/ping6
```

Note : `pkexec` présent mais dépend de la version vulnérable.

---

# SUID / SGID Ubuntu 22.04

Kernel :

```
5.15.0-52-generic
```

## SUID

```
-rwsr-xr-x /usr/bin/chsh
-rwsr-xr-x /usr/bin/pkexec
-rwsr-xr-x /usr/bin/mount
-rwsr-xr-x /usr/bin/chfn
-rwsr-xr-x /usr/bin/su
-rwsr-xr-x /usr/bin/passwd
-rwsr-xr-x /usr/bin/newgrp
-rwsr-xr-x /usr/bin/umount
-rwsr-xr-x /usr/bin/fusermount3
-rwsr-xr-x /usr/bin/gpasswd
-rwsr-xr-x /usr/bin/sudo
-rwsr-xr-x /usr/libexec/polkit-agent-helper-1
-rwsr-xr-- /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x /usr/lib/openssh/ssh-keysign
-rwsr-xr-x /usr/lib/snapd/snap-confine
```

Exemples d'exploits connus :

- **pkexec** → CVE-2021-4034
- **snap-confine** → CVE-2019-7304 (Dirty Sock)

---

## SGID

```
-rwxr-sr-x /usr/sbin/pam_extrausers_chkpwd
-rwxr-sr-x /usr/sbin/unix_chkpwd
-rwxr-sr-x /usr/bin/ssh-agent
-rwxr-sr-x /usr/bin/write.ul
-rwxr-sr-x /usr/bin/expiry
-rwxr-sr-x /usr/bin/crontab
-rwxr-sr-x /usr/bin/wall
-rwxr-sr-x /usr/bin/chage
-rwxr-sr-x /usr/lib/x86_64-linux-gnu/utempter/utempter
```

---

# Rappels importants

Toujours vérifier :

- Version du **kernel**
- Version de **sudo**
- Version de **pkexec**
- Fichiers **SUID**
- **Capabilities**
- **Cron jobs**
- Répertoires **writable**
- **PATH hijacking**
- **Services systemd**
- **Modules kernel**

---

# Outils utiles

| Outil | Description |
|------|-------------|
| LinPEAS | Script d'énumération privesc |
| unix-privesc-check | Vérification automatisée |
| GTFOBins | Exploitation des binaires |
| HackTricks | Documentation offensive |
