# Architecture du Laboratoire

> **Objectif** : Documenter l'architecture technique du laboratoire Purple Team.

---

## 1. Vue d'Ensemble

Le laboratoire repose sur **3 machines virtuelles** interconnectées via un réseau isolé.

```
┌──────────────────┐    ATTAQUE    ┌──────────────────┐
│   KALI           │ ────────────▶ │   DEBIAN-1       │
│   (Attaquant)    │               │   (Victime)      │
│   BeEF           │               │   DVWA           │
│   .10            │               │   .20            │
└──────────────────┘               └──────────────────┘
                                          │
                                          │ LOGS
                                          ▼
                                   ┌──────────────────┐
                                   │   WAZUH          │
                                   │   (SIEM)         │
                                   │   .30            │
                                   └──────────────────┘
```

---

## 2. Détail des Machines

| Machine | Rôle | OS | IP Host-Only | RAM | CPU |
|---|---|---|---|---|---|
| **Kali** | Attaquant | Kali Linux | `192.168.56.10` | 4 Go | 2 |
| **Debian-1** | Victime | Debian 12 | `192.168.56.20` | 4 Go | 2 |
| **Wazuh** | SIEM | Amazon Linux 2023 | `192.168.56.30` | 8 Go | 2 |

---

## 3. Configuration Réseau

### 3.1 — Modes VirtualBox

Chaque VM dispose de **deux adaptateurs réseau** :

| Adaptateur | Mode | Rôle |
|---|---|---|
| **Adaptateur 1** | NAT | Accès Internet (mises à jour) |
| **Adaptateur 2** | Host-Only | Communication du laboratoire |

### 3.2 — Réseau Host-Only

- **Plage** : `192.168.56.0/24`
- **Passerelle** : aucune (réseau isolé)
- **DHCP** : désactivé

### 3.3 — Convention de nommage

Le dernier octet identifie le rôle :
- `.10` → Attaquant
- `.20` → Victime
- `.30` → SIEM

---

## 4. Composants Logiciels

### Kali (Attaquant)
| Outil | Rôle |
|---|---|
| BeEF | Browser Exploitation Framework |
| Firefox | Navigateur de test |

### Debian-1 (Victime)
| Outil | Rôle |
|---|---|
| Docker | Conteneurisation |
| DVWA | Application vulnérable |
| wazuh-agent | Collecte de logs |

### Wazuh (SIEM)
| Composant | Rôle |
|---|---|
| Wazuh Manager | Analyse des événements |
| Wazuh Indexer | Stockage et recherche |
| Wazuh Dashboard | Interface web |

---

## 5. Flux de Données

```
[Kali] ──attaque XSS──▶ [DVWA/Debian-1]
                              │
                              │ écrit dans access.log
                              ▼
                     /var/log/dvwa/access.log
                              │
                              │ lu par l'agent Wazuh
                              ▼
                        [Wazuh Agent]
                              │
                              │ envoie au manager
                              ▼
                     [Wazuh Manager .30]
                              │
                              ▼
                     [Wazuh Dashboard]
```

---

## 6. Sécurité du Laboratoire

| Mesure | Détail |
|---|---|
| Isolation réseau | Host-Only, aucun pont vers le réseau physique |
| Pas d'exposition Internet | VM accessibles uniquement depuis l'hôte |
| Snapshots | Points de restauration |

---

## 7. Prérequis Hôte

| Élément | Minimum | Recommandé |
|---|---|---|
| RAM | 12 Go | 16 Go |
| CPU | 4 cœurs | 8 cœurs |
| Disque | 100 Go | 200 Go |
