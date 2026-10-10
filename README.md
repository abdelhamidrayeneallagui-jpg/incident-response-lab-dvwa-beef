\# Incident Response Lab — DVWA + BeEF + Wazuh



!\[Status](https://img.shields.io/badge/status-completed-success)

!\[Platform](https://img.shields.io/badge/platform-VirtualBox-blue)

!\[SIEM](https://img.shields.io/badge/SIEM-Wazuh-orange)

!\[MITRE](https://img.shields.io/badge/MITRE-T1185-red)



\## 🎯 Objectif



Laboratoire \*\*Purple Team\*\* simulant une attaque \*\*Cross-Site Scripting (XSS)\*\* exploitée via \*\*BeEF\*\*, détectée et analysée avec \*\*Wazuh SIEM\*\*.



Ce projet démontre la chaîne complète :

\*\*Attaque (Red Team)\*\* → \*\*Détection (Blue Team)\*\* → \*\*Réponse à Incident\*\*



\## 🏗️ Architecture



| Machine | Rôle | OS | IP Host-Only |

|---------|------|----|----|

| Kali | Attaquant (BeEF) | Kali Linux | `192.168.56.10` |

| Debian-1 | Victime (DVWA) | Debian 12 | `192.168.56.20` |

| Wazuh | SIEM | Amazon Linux 2023 | `192.168.56.30` |



!\[Architecture](screenshots/01-infrastructure/01-virtualbox-3vm.png)



\## 🛠️ Stack Technique



\- \*\*VirtualBox\*\* — Virtualisation

\- \*\*Docker\*\* — Conteneurisation de DVWA

\- \*\*DVWA\*\* — Application web volontairement vulnérable

\- \*\*BeEF\*\* — Browser Exploitation Framework

\- \*\*Wazuh\*\* — SIEM (Manager + Indexer + Dashboard)

\- \*\*Debian 12 / Kali Linux / Amazon Linux 2023\*\*



\## 🔥 Scénario d'Attaque



1\. L'attaquant (Kali) identifie une faille \*\*XSS Reflected\*\* dans DVWA

2\. Il injecte un \*\*hook BeEF\*\* (`<script src=".../hook.js">`) dans le champ vulnérable

3\. Le navigateur de la victime est \*\*hooké\*\* — BeEF peut exécuter des modules à distance

4\. L'attaque génère des \*\*logs Apache\*\* contenant le payload XSS

5\. \*\*Wazuh détecte l'attaque\*\* via une règle personnalisée (niveau 12)

6\. L'analyste SOC \*\*répond à l'incident\*\* (confinement, éradication, récupération)



\## 📊 Résultats



\- ✅ Détection de l'injection XSS dans les logs Apache

\- ✅ Alerte Wazuh personnalisée (niveau 12)

\- ✅ Mapping MITRE ATT\&CK : \*\*T1185\*\* (Browser Session Hijacking), \*\*T1059.007\*\* (JavaScript)

\- ✅ Rapport d'incident complet (format NIST SP 800-61)

\- ✅ Confinement, éradication et récupération documentés



\## 📁 Structure du Projet

incident-response-lab/
├── README.md
├── LICENSE
├── .gitignore
├── docs/
│ ├── 01-architecture.md
│ ├── 02-setup.md
│ ├── 03-attack-scenario.md
│ ├── 04-detection.md
│ └── 05-incident-report.md
├── detection-rules/
│ └── local_rules.xml
├── scripts/
└── screenshots/
├── 01-infrastructure/
├── 02-config-reseau/
├── 03-dvwa/
├── 04-beef/
├── 05-wazuh/
├── 06-attaque-xss/
└── 07-incident-response/


