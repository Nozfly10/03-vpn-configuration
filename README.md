# 03 — Configuration VPN (OpenVPN & Sophos)

## 🎯 Objectif
Mettre en place des tunnels VPN pour sécuriser les accès distants des utilisateurs et interconnecter des sites distants (VPN site-à-site).

---

## 🏗️ Architecture

```
[Utilisateur distant]                    [Siège]
192.168.100.x                         192.168.1.0/24
      |                                      |
  [OpenVPN Client] <---tunnel chiffré---> [OpenVPN Server]
                                             |
                                      [Réseau interne]
```

---

## ⚙️ Configuration OpenVPN Serveur

### Installation
```bash
sudo apt update && sudo apt install openvpn easy-rsa -y
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
```

### Génération des certificats (PKI)
```bash
# Initialisation
./easyrsa init-pki
./easyrsa build-ca nopass

# Certificat serveur
./easyrsa gen-req serveur nopass
./easyrsa sign-req server serveur

# Certificat client
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1

# Clé Diffie-Hellman
./easyrsa gen-dh
```

### Fichier serveur.conf
```
port 1194
proto udp
dev tun

ca   /etc/openvpn/ca.crt
cert /etc/openvpn/serveur.crt
key  /etc/openvpn/serveur.key
dh   /etc/openvpn/dh.pem

server 10.8.0.0 255.255.255.0
push "route 192.168.1.0 255.255.255.0"
push "dhcp-option DNS 192.168.1.1"

keepalive 10 120
cipher AES-256-CBC
user nobody
group nogroup
persist-key
persist-tun

status  /var/log/openvpn-status.log
verb 3
```

### Démarrage du service
```bash
sudo systemctl start openvpn@serveur
sudo systemctl enable openvpn@serveur
sudo systemctl status openvpn@serveur
```

---

## ⚙️ Configuration Sophos VPN (site-à-site)

```
# Paramètres IKE Phase 1
Authentication : Pre-Shared Key
Encryption     : AES-256
Hash           : SHA-256
DH Group       : Group 14 (2048 bits)
Lifetime       : 28800 s

# Paramètres IPSec Phase 2
Protocol       : ESP
Encryption     : AES-256
Hash           : SHA-256
PFS            : Enabled (Group 14)
Lifetime       : 3600 s
```

---

## ✅ Tests de vérification

```bash
# Vérifier le tunnel
ping 10.8.0.1

# Vérifier les routes
ip route show

# Logs OpenVPN
sudo tail -f /var/log/openvpn-status.log

# Wireshark — filtrer le trafic VPN
udp.port == 1194
```

---

## 🎓 Compétences acquises

- Déploiement d'un serveur OpenVPN
- Gestion d'une PKI (certificats, CA)
- Chiffrement AES-256 et protocole TLS
- Configuration VPN site-à-site Sophos
- Vérification et diagnostic des tunnels VPN
