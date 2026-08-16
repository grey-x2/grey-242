---
title: "Audit WiFi & Serveur DHCP/DNS/NAT avec Kali Linux et Dnsmasq"
date: 2026-08-16
draft: false
categories: ["Cybersécurité", "Réseau", "Infrastructure", "Linux", "Pentest"]
tags: ["wifi", "kali", "pentest", "réseau", "dhcp", "dns", "nat", "dnsmasq", "linux", "infrastructure", "sécurité"]
author: "Marcel BATELA"
language: fr
---

# 📡 Serveur DHCP/DNS/NAT avec Dnsmasq

> *"Un serveur bien configuré est la base d'une infrastructure réseau fiable."*

---

### 🔍 Introduction

La mise en place d'un serveur **DHCP + DNS + NAT** est une compétence essentielle pour tout administrateur réseau. Avec **Dnsmasq**, nous allons créer un serveur tout-en-un léger et performant.

Dans cette deuxième partie, je vous guide pas à pas pour configurer votre propre serveur avec un domaine personnalisé.

---

### 🛠️ Prérequis

- **Linux** (Debian/Ubuntu)
- **Deux interfaces réseau** :
  - `eth0` : Connexion Internet (WAN)
  - `eth1` : Réseau local (LAN)
- **Accès root** (sudo)
- **Connaissances de base** en réseau et Linux

---



### 🔧 Étape 1 : Configuration des interfaces

**Fichier :** `/etc/network/interfaces`

{{< highlight bash >}}
# Interface WAN (eth0) - DHCP
auto eth0
iface eth0 inet dhcp

# Interface LAN (eth1) - Statique
auto eth1
iface eth1 inet static
    address 10.0.0.1
    netmask 255.255.255.0
    network 10.0.0.0
    broadcast 10.0.0.255
{{< /highlight >}}

**Redémarrer les interfaces :**

{{< highlight bash >}}
sudo systemctl restart networking
# ou
sudo ifdown eth0 eth1 && sudo ifup eth0 eth1
{{< /highlight >}}

---

### 📦 Étape 2 : Installation de Dnsmasq

{{< highlight bash >}}
# Mise à jour
sudo apt update

# Installation
sudo apt install dnsmasq -y

# Vérification
dnsmasq --version
{{< /highlight >}}

---

### ⚙️ Étape 3 : Configuration de Dnsmasq

**Sauvegarder la configuration par défaut :**

{{< highlight bash >}}
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.backup
sudo nano /etc/dnsmasq.conf
{{< /highlight >}}

**Configuration complète :**

{{< highlight bash >}}
# ==========================================
# CONFIGURATION DNSMASQ
# Serveur DHCP + DNS + NAT
# Domaine : atte-thm.cg
# ==========================================

# ---------- INTERFACE ----------
interface=eth1
listen-address=10.0.0.1
bind-interfaces

# ---------- DHCP ----------
dhcp-range=10.0.0.10,10.0.0.200,12h
dhcp-option=3,10.0.0.1          # Passerelle
dhcp-option=6,10.0.0.1          # DNS
dhcp-option=15,atte-thm.cg      # Domaine
dhcp-leasefile=/var/lib/misc/dnsmasq.leases
dhcp-authoritative

# ---------- DNS ----------
domain=atte-thm.cg
expand-hosts
local=/atte-thm.cg/

# ---------- Enregistrements DNS ----------
# Serveur
address=/server.atte-thm.cg/10.0.0.1
address=/srv.atte-thm.cg/10.0.0.1

# Services
address=/www.atte-thm.cg/10.0.0.1
address=/mail.atte-thm.cg/10.0.0.1
address=/ftp.atte-thm.cg/10.0.0.1
address=/vpn.atte-thm.cg/10.0.0.1

# Clients statiques (optionnel)
# dhcp-host=00:11:22:33:44:55,client1,10.0.0.10,infinite
# dhcp-host=00:11:22:33:44:66,client2,10.0.0.11,infinite

# ---------- Serveurs DNS externes ----------
no-resolv
server=1.1.1.1          # Cloudflare
server=1.0.0.1          # Cloudflare
server=8.8.8.8          # Google
server=8.8.4.4          # Google

# ---------- Cache ----------
cache-size=1000
neg-ttl=60

# ---------- Logs ----------
log-queries
log-facility=/var/log/dnsmasq.log
log-async

# ---------- Sécurité ----------
domain-needed
bogus-priv
{{< /highlight >}}

**Redémarrer Dnsmasq :**

{{< highlight bash >}}
sudo systemctl restart dnsmasq
sudo systemctl status dnsmasq
{{< /highlight >}}

---

### 🔥 Étape 4 : Configuration du NAT

**Activer le forwarding IP :**

{{< highlight bash >}}
# Temporaire
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Permanent
sudo nano /etc/sysctl.conf
# Ajouter : net.ipv4.ip_forward=1
sudo sysctl -p
{{< /highlight >}}

**Configurer iptables (NAT) :**

{{< highlight bash >}}
# NAT (Masquerade)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# Forwarding
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
{{< /highlight >}}

**Sauvegarder les règles :**

{{< highlight bash >}}
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
{{< /highlight >}}

---

### 🚀 Étape 5 : Tester et vérifier

**Tester le DHCP :**

{{< highlight bash >}}
# Depuis un client
sudo dhclient -v eth1

# Voir les leases
sudo cat /var/lib/misc/dnsmasq.leases
{{< /highlight >}}

**Tester la résolution DNS :**

{{< highlight bash >}}
# Depuis le serveur
nslookup server.atte-thm.cg 10.0.0.1
dig server.atte-thm.cg @10.0.0.1

# Depuis un client
ping -c 4 server.atte-thm.cg
ping -c 4 www.atte-thm.cg
{{< /highlight >}}

**Tester le NAT :**

{{< highlight bash >}}
# Depuis un client
ping -c 4 8.8.8.8
ping -c 4 google.com
curl -I http://google.com
{{< /highlight >}}

---

### 📊 Vérification complète

{{< highlight bash >}}
echo "=== VÉRIFICATION DU SERVEUR ==="

echo -e "\n🔍 Services :"
sudo systemctl status dnsmasq --no-pager

echo -e "\n🔍 Forwarding IP :"
cat /proc/sys/net/ipv4/ip_forward

echo -e "\n🔍 Règles NAT :"
sudo iptables -t nat -L -v | grep MASQUERADE

echo -e "\n🔍 Leases DHCP :"
sudo cat /var/lib/misc/dnsmasq.leases

echo -e "\n🔍 Résolution DNS :"
dig server.atte-thm.cg @10.0.0.1 +short

echo -e "\n🔍 Connectivité Internet :"
ping -c 2 8.8.8.8

echo -e "\n=== VÉRIFICATION TERMINÉE ==="
{{< /highlight >}}

---

### 🔧 Dépannage

**Problème : Le DHCP ne fonctionne pas**

{{< highlight bash >}}
# Vérifier les logs
sudo tail -f /var/log/dnsmasq.log

# Tester la configuration
sudo dnsmasq --test

# Redémarrer
sudo systemctl restart dnsmasq
{{< /highlight >}}

**Problème : Le NAT ne fonctionne pas**

{{< highlight bash >}}
# Vérifier le forwarding
cat /proc/sys/net/ipv4/ip_forward
# Doit afficher 1

# Vérifier les règles
sudo iptables -t nat -L -v
sudo iptables -L FORWARD -v
{{< /highlight >}}

**Problème : DNS ne résout pas**

{{< highlight bash >}}
# Vérifier resolv.conf
cat /etc/resolv.conf

# Tester directement
nslookup google.com 10.0.0.1
{{< /highlight >}}

---

### 🔐 Sécurisation du serveur

**Pare-feu UFW :**

{{< highlight bash >}}
sudo apt install ufw -y

# Autoriser les services
sudo ufw allow 53/tcp   # DNS
sudo ufw allow 53/udp   # DNS
sudo ufw allow 67/udp   # DHCP
sudo ufw allow 68/udp   # DHCP
sudo ufw allow 22/tcp   # SSH

# Activer
sudo ufw enable
{{< /highlight >}}

**Sécuriser SSH :**

{{< highlight bash >}}
sudo nano /etc/ssh/sshd_config
{{< /highlight >}}

{{< highlight bash >}}
Port 2222
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
{{< /highlight >}}

{{< highlight bash >}}
sudo systemctl restart sshd
{{< /highlight >}}


---

### 📚 Pour aller plus loin (Serveur)

- 🔒 Ajouter un serveur **VPN** (WireGuard/OpenVPN)
- 📊 Mettre en place un **monitoring** (Grafana/Prometheus)
- 🛡️ Configurer un **pare-feu avancé**
- 📝 Mettre en place des **backups**

---

### 📡 Ressources (Serveur)

- [Documentation Dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html)
- [Guide iptables](https://netfilter.org/documentation/)
- [Sécurisation Linux](https://www.linuxfoundation.org/)

---

## 🎯 Conclusion

Vous avez maintenant les compétences pour :

✅ **Auditer** vos réseaux WiFi avec Kali Linux  
✅ **Sécuriser** vos réseaux sans fil  
✅ **Configurer** un serveur DHCP + DNS + NAT avec Dnsmasq  
✅ **Tester** et **dépanner** votre infrastructure  

---

> *"La sécurité et l'infrastructure sont les piliers d'un système fiable."* 🔒



*Merci d'avoir lu cet article ! N'hésitez pas à partager vos retours.* 🔒