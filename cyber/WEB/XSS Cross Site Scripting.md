# XSS Cheatsheet

---

## 🧠 Définition

Le **JavaScript est exécuté dans le navigateur de la victime**, ce qui permet de prendre le contrôle de l’application.

➡️ En théorie, les frameworks modernes incluent des protections par défaut  
➡️ En pratique, des vulnérabilités existent encore fréquemment

---

## ⚡ Types de XSS

### 1. Reflected XSS
- Injecté via une requête (URL, paramètre GET/POST)
- Réfléchi directement dans la réponse
- ⚠️ Limité (souvent nécessite interaction utilisateur)

---

### 2. Stored XSS
- Payload stocké côté serveur (DB, logs, etc.)
- Exécuté automatiquement lorsqu’un utilisateur consulte la donnée

💡 Exemple :
- Plugin WordPress qui log :
  - IP
  - User-Agent
- On modifie le header `User-Agent` avec du JS
- Quand l’admin ouvre le dashboard → 💥 XSS exécuté

---

### 3. DOM-based XSS
- Tout se passe côté client (JavaScript)
- Aucune interaction serveur nécessaire

---

## 🖼️ Schémas

### Reflected XSS
![Reflected XSS](./images/reflected.png)

---

### Stored XSS
![Stored XSS](./images/stored.png)

---

### DOM-based XSS
![DOM XSS](./images/dom.png)

---

## 🔁 Exemple DOM XSS

Redirection vers une URL :

```html
<img src=x onerror="window.location.href='https://attacker.com'">
```

---

## 🧪 Méthodologie de test

### 1. Tester injection HTML

```html
<h1>test</h1>
```

➡️ Permet de vérifier si :
- Le HTML est interprété
- Un filtrage est en place

---

### 2. Tester injection JavaScript

Exemples simples :

```html
<script>alert(1)</script>
```

```html
<img src=x onerror=alert(1)>
```

---

## 🌐 Navigation & Isolation

### Firefox Containers

- Permet d’isoler les cookies par contexte
- Exemple :
  - Cookie créé dans conteneur 1
  - ❌ Non accessible dans conteneur 2

➡️ Utile pour tests multi-session / multi-user

---

## 🍪 Cookies & XSS

Les cookies servent à stocker :
- état de session
- informations utilisateur

---

### Flags importants

#### 🔐 Secure
- Cookie envoyé uniquement via HTTPS
- Protège contre interception réseau

---

#### 🚫 HttpOnly
- Empêche JavaScript d’accéder au cookie

➡️ Si **absent** :
```javascript
document.cookie
```
➡️ possible → **vol de session via XSS**

---

### ⚠️ Important

- Si `HttpOnly` est activé → JS ne peut pas lire le cookie
- Si `Secure` absent → risque interception réseau

---

## 📚 Ressources

- https://www.w3schools.com/
- https://portswigger.net/web-security/cross-site-scripting

---

## 🧠 Rappels importants

- Toujours tester **HTML injection avant JS**
- Identifier le type de XSS :
  - Reflected
  - Stored
  - DOM
- Vérifier les protections :
  - Encodage
  - CSP
  - HttpOnly / Secure
- Tester plusieurs payloads

---

## 🔚 Conclusion

XSS reste une vulnérabilité :
- **très commune**
- **très puissante**

➡️ Peut mener à :
- vol de session
- escalade de privilèges
- takeover de compte

---

💡 Astuce : toujours penser **"où est injectée la donnée et comment elle est interprétée ?"**
