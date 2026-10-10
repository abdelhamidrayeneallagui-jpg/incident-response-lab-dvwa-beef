# Guide d'Installation

> **Objectif** : Permettre la reproduction complète du laboratoire.

---

## 1. Prérequis

- VirtualBox 7.x installé
- 12 Go de RAM minimum (16 Go recommandé)
- 100 Go d'espace disque
- 3 ISO : Kali Linux, Debian 12, Wazuh OVA

---

## 2. Création du Réseau Host-Only

1. VirtualBox → **Fichier** → **Outils** → **Gestionnaire de réseau**
2. Onglet **Réseaux Host-Only** → **Créer**
3. Configurer :
   - **Adresse IPv4** : `192.168.56.1`
   - **Masque** : `255.255.255.0`
   - **DHCP** : désactivé

---

## 3. Création des VM

| VM | OS | RAM | Disque |
|---|---|---|---|
| Kali | Kali Linux | 4 Go | 40 Go |
| Debian-1 | Debian 12 | 4 Go | 50 Go |
| Wazuh | Import OVA | 8 Go | 50 Go |

**Réseau** pour chaque VM :
- Adaptateur 1 : **NAT**
- Adaptateur 2 : **Host-Only**

---

## 4. Configuration Réseau

### Kali (`192.168.56.10`)
```bash
sudo nmcli connection modify "Wired connection 2" ipv4.addresses 192.168.56.10/24
sudo nmcli connection modify "Wired connection 2" ipv4.method manual
sudo nmcli connection down "Wired connection 2"
sudo nmcli connection up "Wired connection 2"
```

### Debian-1 (`192.168.56.20`)
```bash
sudo nmcli connection modify "Wired connection 1" ipv4.addresses 192.168.56.20/24
sudo nmcli connection modify "Wired connection 1" ipv4.method manual
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"
```

### Wazuh (`192.168.56.30`)
```bash
sudo tee /etc/systemd/network/10-eth1-static.network << 'EOF'
[Match]
Name=eth1

[Network]
DHCP=no
Address=192.168.56.30/24
Gateway=
DNS=8.8.8.8
EOF

sudo systemctl restart systemd-networkd
```

---

## 5. Installation de DVWA

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker

sudo mkdir -p /var/log/dvwa
sudo chmod 755 /var/log/dvwa

sudo docker run -d --name dvwa --restart unless-stopped \
  -p 80:80 \
  -v /var/log/dvwa:/var/log/apache2 \
  vulnerables/web-dvwa
```

**Accès** : `http://192.168.56.20` (admin / password)

---

## 6. Installation de Wazuh (OVA)

1. Importer l'OVA dans VirtualBox
2. Configurer l'adaptateur Host-Only
3. Démarrer la VM
4. Dashboard : `https://192.168.56.30` (admin / admin)

---

## 7. Installation de l'Agent Wazuh

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | \
  gpg --dearmor -o /usr/share/keyrings/wazuh.gpg
chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | \
  tee /etc/apt/sources.list.d/wazuh.list
apt update

WAZUH_MANAGER='192.168.56.30' apt install -y wazuh-agent
sed -i 's/MANAGER_IP/192.168.56.30/g' /var/ossec/etc/ossec.conf

systemctl enable wazuh-agent
systemctl start wazuh-agent
```

---

## 8. Installation de BeEF

```bash
sudo apt install -y beef-xss
sudo nano /etc/beef-xss/config.yaml
# Modifier :
#   host: "0.0.0.0"
#   public: "192.168.56.10"
#   passwd: "VotreMotDePasse"

sudo systemctl start beef-xss
```

---

## 9. Validation

| Test | Commande | Attendu |
|---|---|---|
| Ping inter-VM | `ping 192.168.56.30` | Réponse |
| DVWA | `curl http://192.168.56.20` | `302 Found` |
| BeEF | `curl http://192.168.56.10:3000/hook.js` | JavaScript |
| Agent | Dashboard → Agents | `Active` |
