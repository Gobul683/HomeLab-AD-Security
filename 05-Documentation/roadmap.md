# 🗺️ Roadmap du Projet – Home Lab AD & Sécurité

Ce projet suit une méthodologie progressive orientée infrastructure et cybersécurité :

**Construire → Attaquer → Durcir**

---

## ✅ Phase 1 – Infrastructure Core (Terminée)

Objectif : mettre en place une infrastructure d’entreprise fonctionnelle et segmentée.

- Déploiement de l’hyperviseur Proxmox VE
- Organisation des machines virtuelles (Resource Pools)
- Mise en place du pare-feu et du routage avec pfSense
- Création du domaine Active Directory
- Configuration des services DNS et DHCP
- Intégration des postes clients Windows au domaine

---

## 🚧 Phase 2 – Sécurité Offensive & Audit (En cours)

Objectif : analyser la surface d’attaque de l’Active Directory avant durcissement.

- Déploiement d’une machine Kali Linux
- Reconnaissance réseau (Nmap)
- Attaques sur la résolution de noms (LLMNR, mDNS, NBT-NS)
- Capture de hashs NTLMv2
- Cassage de mots de passe (John the Ripper)
- Analyse des failles par défaut de l’AD

---

## 📅 Phase 3 – Hardening & Remédiation (En cours)

Objectif : réduire la surface d’attaque et sécuriser l’environnement.

- Désactivation de LLMNR via GPO
- Désactivation de mDNS via GPO
- Désactivation de NetBIOS au niveau des postes clients
- Nettoyage des caches réseau
- Validation finale (attacks blocked)

---

## 🔄 Améliorations futures

- Mise en place d’une DMZ sécurisée
- Segmentation réseau avancée
- Journalisation et supervision
- Automatisation des tâches de sécurité (PowerShell)
