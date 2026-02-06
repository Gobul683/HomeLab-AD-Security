# 🖥️ Virtualisation & Organisation – Proxmox VE

> **Couche de virtualisation du home lab**
>
> Cette page décrit comment Proxmox est utilisé pour structurer,
> isoler et sécuriser l’infrastructure avant toute configuration
> réseau, Active Directory ou sécurité.

---

## 🎯 Objectifs de cette couche

- Isoler les environnements (serveurs, utilisateurs, DMZ)
- Reproduire une architecture d’entreprise réaliste
- Préparer les scénarios Red Team / Blue Team
- Éviter tout impact sur l’environnement personnel

## 1. Organisation & Virtualisation (Proxmox)

Afin de ne pas impacter les services de production personnelle, j'ai créé un cloisonnement logique via des **Resource Pools**.

* **Pool `LAB-CYBER-AD` :** Regroupe toutes les VMs du projet.
* **Sous-Pools :** SERVERS, USERS, DMZ, MGMT, BACKUP.

<img width="370" height="88" alt="Resource Pool" src="https://github.com/user-attachments/assets/bc20bc3e-0c3b-4115-ad99-328744bf70be" />
<img width="137" height="148" alt="Structure Dossiers" src="https://github.com/user-attachments/assets/bc577332-e2d3-4785-91be-33ba5d40326b" />

**Inventaire des Machines Virtuelles :**
* **🔥 pfSense :** Routeur/Pare-feu (Cœur du réseau).
* **🛡️ SRV-AD :** Windows Server 2019 (Contrôleur de Domaine).
* **🖥️ PC-Win10 :** Client Utilisateur.
* **🌐 SRV-WEB :** (Prévu) Serveur Web en DMZ.

<img width="222" height="163" alt="VM Inventory" src="https://github.com/user-attachments/assets/e02968d7-6082-45cd-9c9a-c3bdcfdb05d4" />

