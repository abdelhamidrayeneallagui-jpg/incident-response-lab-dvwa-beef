# Ingénierie de Détection — Wazuh

> **Objectif** : Documenter la règle Wazuh personnalisée pour détecter BeEF.

---

## 1. Contexte

Wazuh a des règles génériques (31100, niveau 6 "A web attack") mais rien de **spécifique** à BeEF.

**Objectif** : règle qui détecte `hook.js`, déclenche niveau 12, mappe sur MITRE T1185.

---

## 2. Règle Personnalisée

Fichier : [`detection-rules/local_rules.xml`](../detection-rules/local_rules.xml)

```xml
<group name="web,appsec,xss,beef,">
  <rule id="100100" level="12">
    <if_sid>31100</if_sid>
    <url>hook.js</url>
    <description>Possible BeEF hook injection detected - Browser Session Hijacking</description>
    <mitre><id>T1185</id></mitre>
    <group>beef_hook,</group>
  </rule>
</group>
```

---

## 3. Explication

| Balise | Valeur | Signification |
|---|---|---|
| id | 100100 | Plage utilisateur (100000-120000) |
| level | 12/15 | Sévérité CRITIQUE |
| if_sid | 31100 | Règle Apache parente |
| url | hook.js | Cherche dans le champ url décodé |
| mitre.id | T1185 | Browser Session Hijacking |

---

## 4. Test

Utilisation de `wazuh-logtest` avec un vrai log Apache.

**Résultat** :
```
**Phase 3: Completed filtering (rules).
        id: '100100'
        level: '12'
        description: 'Possible BeEF hook injection detected'
        mitre.id: ['T1185']
**Alert to be generated.
```

✅ La règle fonctionne.

---

## 5. Validation en Production

Alerte visible dans Wazuh Discover :

| Champ | Valeur |
|---|---|
| rule.id | 100100 |
| rule.level | 12 |
| data.srcip | 192.168.56.10 |
| data.url | /vulnerabilities/xss_r/?name=...hook.js... |

---

## 6. Déploiement

```bash
nano /var/ossec/etc/rules/local_rules.xml
xmllint --noout /var/ossec/etc/rules/local_rules.xml
/var/ossec/bin/wazuh-control restart
grep "Total rules" /var/ossec/logs/ossec.log | tail -1
```

---

## 7. Références

- [Wazuh Rules Syntax](https://documentation.wazuh.com/current/user-manual/ruleset/ruleset-xml-syntax/rules.html)
- [MITRE ATT&CK T1185](https://attack.mitre.org/techniques/T1185/)
