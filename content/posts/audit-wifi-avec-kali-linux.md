---
title: "Audit WiFi avec Kali Linux : Guide Pratique"
date: 2026-08-16
draft: false
categories: ["Cybersécurité", "Pentest"]
tags: ["wifi", "kali", "pentest", "sécurité", "réseau"]
author: "Marcel BATELA"
language: fr
---

# 📡 Audit WiFi avec Kali Linux

> *"La sécurité d'un réseau sans fil commence par sa compréhension."*

---

## 🔍 Introduction

L'**audit WiFi** est une étape essentielle pour sécuriser vos réseaux sans fil. Avec **Kali Linux**, nous allons explorer les techniques d'analyse et de sécurisation des réseaux WiFi.

Dans cet article, je vous partage les bases de l'audit WiFi avec les outils les plus utilisés par les professionnels de la cybersécurité.

---

## 🛠️ Prérequis

- **Kali Linux** installé (ou en live USB)
- **Carte WiFi** compatible avec le mode monitor (ex: Alfa AWUS036ACH)
- **Connaissances de base** en réseau et Linux

---

## 🔧 Installation des outils

Kali Linux intègre déjà la plupart des outils nécessaires. Voici les principaux :

{{< highlight bash >}}
# Vérifier les outils installés
airmon-ng --version
airodump-ng --version
aireplay-ng --version

# Mettre à jour les outils
sudo apt update && sudo apt upgrade -y
{{< /highlight >}}

---

## 📡 Phase 1 : Mise en mode monitor

La première étape consiste à passer votre carte WiFi en mode monitor :

{{< highlight bash >}}
# Lister les interfaces réseau
sudo iwconfig

# Démarrer le mode monitor
sudo airmon-ng start wlan0

# Vérifier que l'interface est en mode monitor
sudo iwconfig wlan0mon
{{< /highlight >}}

---

## 📊 Phase 2 : Scan des réseaux

Une fois en mode monitor, nous allons scanner les réseaux disponibles :

{{< highlight bash >}}
# Démarrer le scan
sudo airodump-ng wlan0mon

# Pour capturer plus d'informations sur un réseau spécifique
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w capture wlan0mon
{{< /highlight >}}

**Explication des options :**
- `-c 6` : Canal du réseau cible
- `--bssid` : Adresse MAC du routeur
- `-w capture` : Nom du fichier de capture

---

## 🎯 Phase 3 : Capture du handshake WPA/WPA2

Pour analyser un réseau sécurisé, nous devons capturer le **handshake WPA/WPA2** :

{{< highlight bash >}}
# Capturer le handshake
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 -w capture wlan0mon

# Déauthentifier un client pour forcer la reconnexion
sudo aireplay-ng -0 2 -a 00:11:22:33:44:55 wlan0mon
{{< /highlight >}}

> ⚠️ **Important :** Cette étape est légale uniquement sur vos propres réseaux ou avec autorisation explicite.

---

## 🔐 Phase 4 : Sécurisation

Voici les **bonnes pratiques** pour sécuriser votre réseau WiFi :

### ✅ Recommandations essentielles

| Recommandation | Détail |
|----------------|--------|
| **WPA3** | Utiliser WPA3 si disponible |
| **Mot de passe fort** | Minimum 12 caractères avec symboles |
| **Désactiver WPS** | Vulnérabilité connue |
| **Changer le SSID** | Éviter le nom par défaut |
| **Mise à jour firmware** | Maintenir le routeur à jour |

### 🛡️ Vérification de la sécurité

{{< highlight bash >}}
# Vérifier les vulnérabilités de votre réseau
sudo wash -i wlan0mon  # Détecter les WPS actifs

# Analyser les clients connectés
sudo airodump-ng -c 6 --bssid 00:11:22:33:44:55 wlan0mon
{{< /highlight >}}

---

## 🎯 Conclusion

L'audit WiFi avec Kali Linux est une compétence essentielle pour tout professionnel de la **cybersécurité**. En comprenant les vulnérabilités des réseaux sans fil, vous pouvez mieux les protéger.

### 📚 Pour aller plus loin

- 📖 **The Web Application Hacker's Handbook** - Dafydd Stuttard
- 🏆 **HackTheBox** - Challenges WiFi
- 🎥 **YouTube** - Tutoriels pratiques

---

## 📡 Ressources

- [Kali Linux Documentation](https://www.kali.org/docs/)
- [Aircrack-ng Guide](https://www.aircrack-ng.org/)
- [OWASP WiFi Security](https://owasp.org/)

---

> *"La sécurité n'est pas un produit, c'est un processus."* — Bruce Schneier

---

*Merci d'avoir lu cet article ! N'hésitez pas à partager vos retours.* 🔒