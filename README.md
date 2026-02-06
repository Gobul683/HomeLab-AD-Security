📌 Projet personnel réalisé dans le cadre d'une formation Administrateur
Infrastructure Sécurisée (niveau 6 – Bac+4).

# 🧠 Home Lab – Active Directory & Sécurité

## 🎯 Objectif
Simuler une infrastructure d’entreprise sécurisée afin de pratiquer
l’administration systèmes, réseaux et la cybersécurité (offensive & défensive).

## 🧰 Stack technique
- Proxmox VE (virtualisation)
- pfSense (pare-feu, routage, VLAN)
- Windows Server (AD DS, DNS, DHCP, GPO)
- Windows 10 (clients)
- Kali Linux (audit & attaques)
- PowerShell / Bash

## 🧪 Méthodologie
Construire → Attaquer → Durcir

## 📂 Accès à la documentation
- 📐 [01 – Architecture](01-Architecture/)
- 🏗️ [02 – Infrastructure](02-Infrastructure/)
- 🔐 [03 – Sécurité (Red & Blue Team)](03-Security/)
  - ⚔️ [Red Team – Attaques Active Directory](03-Security/red-team.md)
  - 🛡️ [Blue Team – Hardening & Remédiation](03-Security/blue-team-hardening.md)
- 🗺️ [05 – Documentation & Roadmap](05-Documentation/)

# 🏰 Projet Home Lab : Infrastructure Active Directory & Sécurité Offensive

![Status](https://img.shields.io/badge/Status-En%20Cours-yellow) ![Tech](https://img.shields.io/badge/Tech-Proxmox%20|%20pfSense%20|%20Windows%20Server-blue) ![Security](https://img.shields.io/badge/Security-Red%20Teaming-red)

## 🎯 Objectif du projet

Ce lab a pour but de simuler un environnement d'entreprise complet pour maîtriser la chaîne de défense et d'attaque. Il permet de mettre en pratique :
* **La Virtualisation** (Proxmox VE, vSwitching).
* **L'Infrastructure** (Active Directory, DNS, DHCP, Routage).
* **La Cybersécurité** (Attaque et Défense, GPO, Hardening).

---

## 🗺️ Roadmap du Projet (Cycle de vie)

Ce projet suit une méthodologie progressive : **Construire, Attaquer, Durcir.**

* ✅ **Phase 1 : Infrastructure Core (Terminée)**
    * Déploiement de l'hyperviseur et segmentation réseau.
    * Mise en place du routeur/pare-feu (pfSense).
    * Création du Domaine (AD DS) et intégration des clients.
* 🚧 **Phase 2 : Offensive Security & Audit (En cours)**
    * *Pourquoi maintenant ?* Avant de sécuriser, il faut comprendre les failles.
    * Déploiement de **Kali Linux**.
    * Scan de vulnérabilités et attaques AD (LLMNR, Kerberoasting).
* 📅 **Phase 3 : Hardening & Architecture Avancée (À venir)**
    * Mise en place d'une DMZ et ségrégation stricte.
    * Application des correctifs basés sur les résultats de la Phase 2.

---

## 📑 Table des Matières

1.  [Organisation & Virtualisation (Proxmox)](#1-organisation--virtualisation-proxmox)
2.  [Architecture Réseau & Segmentation](#2-architecture-réseau--segmentation)
3.  [Routage & Pare-Feu (pfSense)](#3-routage--pare-feu-pfsense)
4.  [Mise en place de l'Active Directory](#4-mise-en-place-de-lactive-directory)
5.  [Intégration du Client Windows 10](#5-intégration-du-client-windows-10)
6.  [Phase 2 : Sécurité Offensive (Red Teaming)](#-phase-2--sécurité-offensive-red-teaming)
7.  [Phase 3 : Hardening & Remédiation (Blue Team)](#-phase-3--hardening--remédiation-blue-team)

---

> ✅ Astuce lecture : tu peux tout lire d’un coup en scrollant, **ou** utiliser les sections repliables ci-dessous pour naviguer plus vite.

---

<details open>
<summary><strong>🧱 1. Organisation & Virtualisation (Proxmox)</strong></summary>

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

</details>

---

<details open>
<summary><strong>🌐 2. Architecture Réseau & Segmentation</strong></summary>

Application du principe de **Défense en Profondeur** via une segmentation réseau stricte (vSwitchs).

| Interface | Zone | Rôle | Description |
| :--- | :--- | :--- | :--- |
| `vmbr0` | **WAN** | Internet | Pont vers la Box physique. |
| `vmbr1` | **SERVERS** | Infra | Héberge l'AD, DNS, DHCP. |
| `vmbr2` | **LAN_USERS** | Clients | Zone des postes de travail. |
| `vmbr3` | **DMZ** | Public | Services exposés (Web). |
| `vmbr4` | **MGMT** | Admin | Gestion hors-bande (Out-of-Band). |
| `vmbr5` | **BACKUP** | Sauvegarde | Trafic dédié aux backups (Veeam/PBS). |

**Preuve de configuration (Proxmox Network) :**
<img width="2542" height="426" alt="Network Config" src="https://github.com/user-attachments/assets/14219d87-ed37-43da-99ce-fc591a517881" />

</details>

---

<details open>
<summary><strong>🔥 3. Routage & Pare-Feu (pfSense)</strong></summary>

### 3.1. Configuration des Interfaces
Utilisation de cartes virtuelles **Intel E1000** pour une compatibilité native maximale.

* **WAN** -> `em0`
* **LAN (Servers)** -> `em1` (`192.168.10.254`)
* **OPT1 (Users)** -> `em2` (`192.168.20.254`)

<img width="1558" height="309" alt="pfSense Interfaces" src="https://github.com/user-attachments/assets/2cfbf98d-0a9f-47b2-a009-1aa2e76fe825" />

### 3.2. Services (DHCP & DNS)
Configuration du DHCP sur la zone USERS (`192.168.20.x`) avec injection des DNS publics (`1.1.1.1`, `8.8.8.8`) pour l'accès internet initial.

<img width="1072" height="253" alt="DHCP Config" src="https://github.com/user-attachments/assets/38151383-9ff6-4076-b430-1270d0f3873d" />
<img width="1077" height="373" alt="DNS Config" src="https://github.com/user-attachments/assets/d4a97b5a-4a6e-48ae-9615-049598428b66" />

### 3.3. Règles de Pare-Feu
Ouverture des flux sortants pour la zone USERS (Allow Any) afin de permettre l'installation et les mises à jour.  
*Note : Ces règles seront durcies dans la Phase 3.*

<img width="1095" height="360" alt="Firewall Rules" src="https://github.com/user-attachments/assets/c73b7fa7-3d97-4840-bb65-465b7c2216ea" />

### 3.4. Initialisation WAN
Décochage des options **Block RFC1918** pour autoriser le trafic entrant depuis la Box Internet (Double NAT).

<img width="1072" height="335" alt="WAN Settings" src="https://github.com/user-attachments/assets/f9160e32-4a1c-4cad-8cfa-6214de94ce8f" />

</details>

---

<details open>
<summary><strong>🛡️ 4. Mise en place de l'Active Directory</strong></summary>

### 4.1. Préparation du Serveur (SRV-AD)
* **OS :** Windows Server 2019
* **IP Fixe :** `192.168.10.10` / `24`
* **DNS Préféré :** `127.0.0.1` (Lui-même)

<img width="530" height="564" alt="Static IP" src="https://github.com/user-attachments/assets/2c11365b-17dc-41c2-97c8-6c1e1aa02809" />
<img width="850" height="330" alt="Hostname" src="https://github.com/user-attachments/assets/0d09ae82-d1c5-4297-b466-7826cd46eb9c" />

### 4.2. Promotion (dcpromo)
Installation du rôle **AD DS** et création de la nouvelle forêt **`lab.lan`**.

<img width="340" height="202" alt="AD DS Role" src="https://github.com/user-attachments/assets/8d131fea-dcb0-4cdf-a438-cd58f545e04c" />
<img width="843" height="613" alt="Install Wizard" src="https://github.com/user-attachments/assets/d7c501a4-6de7-4565-8cb9-add533129e76" />
<img width="825" height="602" alt="Forest Creation" src="https://github.com/user-attachments/assets/f73b4a38-6983-494e-8c81-f1ca22bf0dc2" />

### 4.3. Gestion des Identités
Création d'utilisateurs standards pour éviter l'usage du compte Administrateur au quotidien.
* **User :** Musti Ugur (`mugur`)
* **Groupe :** Utilisateurs du domaine

<img width="472" height="408" alt="User Creation" src="https://github.com/user-attachments/assets/f93168ba-9eca-45d4-90d9-cc5a3abdba0f" />

> [!WARNING]
> **🛡️ Note de Sécurité : Le Principe du Moindre Privilège**
> Il est impératif de ne jamais utiliser le compte "Administrateur" pour une session utilisateur standard. Si ce compte est compromis (Drive-by download, Phishing), l'attaquant obtient immédiatement le contrôle total du domaine.

</details>

---

<details open>
<summary><strong>🖥️ 5. Intégration du Client Windows 10</strong></summary>

### 5.1. Configuration DNS
Le point critique : le client doit impérativement utiliser l'IP du serveur AD (`192.168.10.10`) comme serveur DNS pour résoudre le domaine `lab.lan`.

<img width="655" height="236" alt="Client Network" src="https://github.com/user-attachments/assets/87029898-93e2-4614-bbb2-7c1ce3b4b9f4" />
<img width="455" height="520" alt="Client DNS" src="https://github.com/user-attachments/assets/89f4ec16-651b-4530-832f-d6b13a26b7f7" />

### 5.2. Jonction au Domaine
Connexion réussie au domaine `lab.lan` et authentification avec le compte Administrateur pour autoriser la jonction.

<img width="514" height="368" alt="Domain Join" src="https://github.com/user-attachments/assets/2859cb07-6bfa-4d2d-a315-627111f57c9e" />
<img width="362" height="170" alt="Welcome Msg" src="https://github.com/user-attachments/assets/30520712-68e1-4e27-add9-6840619ef6bf" />

### 5.3. Validation Finale
Ouverture de session avec l'utilisateur `LAB\mugur` et vérification via l'invite de commande (`whoami`, `ipconfig`).

<img width="771" height="617" alt="Final Validation" src="https://github.com/user-attachments/assets/91803fd8-3e14-44a6-9c46-be027c0dbfd4" />

</details>

---

## 6. Prochaines Étapes : Offensive Security

L'infrastructure "Core" est fonctionnelle. Le projet bascule maintenant en **Phase 2 : Attaque**.

---

# ⚔️ Phase 2 : Sécurité Offensive (Red Teaming)

Maintenant que l'infrastructure est fonctionnelle, j'ai basculé en mode "Attaquant" (depuis la machine Kali Linux dans la zone USERS) pour auditer la sécurité de la configuration par défaut de l'Active Directory.

## 1. Reconnaissance & Cartographie
L'objectif est d'identifier les machines vivantes et les services exposés sans connaitre l'architecture au préalable.

* **Outil utilisé :** `Nmap`
* **Commande :** `nmap -sV -O 192.168.10.10`
* **Résultats :**
    * Découverte des ports critiques ouverts : **88** (Kerberos), **389** (LDAP), **445** (SMB).
    * Confirmation qu'il s'agit d'un Contrôleur de Domaine Windows Server.

<img width="736" height="664" alt="Capture d'écran 2026-01-24 164232" src="https://github.com/user-attachments/assets/59f836b7-528b-4bef-a09b-286cf9a13f00" />

## 2. Attaque Man-in-the-Middle (LLMNR Poisoning)
Windows utilise par défaut le protocole **LLMNR** (Link-Local Multicast Name Resolution) pour chercher des noms de machines sur le réseau local s'il ne les trouve pas via le DNS.

* **Scénario :** Un utilisateur fait une erreur de frappe en cherchant un dossier partagé (ex: `\\serveur-inconnu`).
* **L'attaque :**
    * L'outil **Responder** écoute le réseau.
    * Lorsqu'il entend la requête LLMNR, il répond "C'est moi le serveur !".
    * La victime envoie alors son **Hash NTLMv2** (son mot de passe chiffré) à l'attaquant.

<img width="1453" height="798" alt="Capture d'écran 2026-01-24 165045" src="https://github.com/user-attachments/assets/ac1bfff9-40a3-4b13-a897-7746a6811c2d" />

## 3. Cassage de Mot de Passe (Cracking)
Une fois le hash NTLMv2 récupéré, il est inexploitable tel quel pour se connecter. Il faut le "casser" pour retrouver le mot de passe en clair.

* **Outil utilisé :** `John the Ripper`
* **Méthode :** Attaque par dictionnaire (Wordlist).
* **Résultat :** Le mot de passe de l'utilisateur `mugur` a été retrouvé.

> **Preuve de concept :**
> `mugur:rocknroll25!` (Le mot de passe a été cracké avec succès).

<img width="880" height="310" alt="Capture d'écran 2026-01-24 170737" src="https://github.com/user-attachments/assets/ebf337cc-0eeb-440e-beba-547ec733c645" />

---

# 🛡️ Phase 3 : Hardening & Remédiation (Blue Team)

### 🚨 Analyse de l'incident (Post-Mortem)
Lors de la phase d'attaque (Phase 2), la capture d'écran de **Responder** a révélé que notre machine Windows 10 "bavardait" imprudemment sur le réseau via trois protocoles distincts pour tenter de résoudre des noms de machines :
1.  **LLMNR** (Link-Local Multicast Name Resolution).
2.  **mDNS** (Multicast DNS).
3.  **NBT-NS** (NetBIOS Name Service).

C'est cette configuration par défaut qui a permis à l'attaquant d'empoisonner les réponses et de voler les identifiants NTLMv2. Pour sécuriser le réseau, nous devons neutraliser ces trois vecteurs un par un.

---

## 1. Neutralisation de LLMNR (Via GPO)

* **Action :** Modification de la *Default Domain Policy* pour interdire la "Résolution de noms multidiffusion".  
  > *Note : Dans un environnement de production réel, il est recommandé de créer une GPO dédiée pour la sécurité. Pour ce lab, nous modifions la stratégie par défaut par souci de simplicité.*

* **Chemin GPO :** `Configuration ordinateur > Modèles d'administration > Réseau > Client DNS`.
* **Paramètre :** *Désactiver la résolution de noms multidiffusion*.

<img width="1465" height="867" alt="Capture d'écran 2026-01-24 180429" src="https://github.com/user-attachments/assets/329cf28d-bba9-4b71-b128-b9c0091b2595" /> 

* **Commande d'application :** `gpupdate /force` (dans une invite de commande).

> [!NOTE]
> **Subtilité technique (Logique GPO) :**
> Le paramètre se nomme *"Désactiver la résolution de noms multidiffusion"*.
> Il faut donc le mettre sur **✅ Activé (Enabled)** pour que l'interdiction prenne effet. (On active la désactivation).

<img width="321" height="145" alt="image" src="https://github.com/user-attachments/assets/a93b1e02-63b2-41ec-8dcd-6c471c0298a5" />

* **Résultat :** Responder ne capture plus de trafic étiqueté `[LLMNR]`.

---

## 2. La persistance du mDNS (Noms longs)

Lors d'un test avec un nom de domaine complet (`\\test-apres-desactivation.LLMNR`), j'ai remarqué que le protocole **mDNS** prenait le relais.  
Contrairement au LLMNR, un simple `gpupdate` ne suffit pas toujours à arrêter ce service : un redémarrage est nécessaire.

<img width="1443" height="731" alt="Capture d'écran 2026-01-24 181014" src="https://github.com/user-attachments/assets/513cc2e7-d1fa-426a-95a5-bdb769a1abe5" />

* **Action :** Modification de la GPO pour cibler spécifiquement *Configurer le protocole DNS de multidiffusion*.
* **Chemin GPO :** Identique au précédent.
* **Paramètre :** *Configurer le protocole DNS de multidiffusion (mDNS)*.

<img width="1460" height="858" alt="Capture d'écran 2026-01-24 182049" src="https://github.com/user-attachments/assets/1b8e332a-488d-45aa-851e-de22eb348c4f" />

* **Valeur :** 🚫 **Désactivé**.

> [!WARNING]
> **Attention à la logique inversée :**
> Contrairement au paramètre LLMNR précédent, ici l'intitulé est *"Configurer..."*.
> Nous choisissons donc **Désactivé** pour dire "Je ne veux pas de cette configuration". (On désactive la fonctionnalité).

<img width="400" height="145" alt="image" src="https://github.com/user-attachments/assets/97aa7171-ed23-4603-8473-903847b2ae1f" />

* **Application :** **Redémarrage obligatoire** du poste client pour purger le service *Dnscache*.

**Test intermédiaire :**
En tentant d'accéder à `\\test-apres-configuration-mDNS`, Windows affiche désormais un message d'erreur réseau immédiat. C'est le comportement attendu.

<img width="247" height="34" alt="Capture d'écran 2026-01-24 182501" src="https://github.com/user-attachments/assets/790a785e-2c21-48f6-9492-5e82e662ea12" />
<img width="609" height="183" alt="Capture d'écran 2026-01-24 183230" src="https://github.com/user-attachments/assets/c4d05889-f655-4f6a-83f8-ffbce4993ab5" />

---

## 3. La surprise du NetBIOS (Noms courts)

Pensant être sécurisé, j'ai réalisé un dernier test avec un nom court : `\\test`.  
**Surprise :** Responder s'est réveillé et a capturé des hashs via **NBT-NS** (NetBIOS) !

<img width="1438" height="691" alt="Capture d'écran 2026-01-24 183407" src="https://github.com/user-attachments/assets/704e2a5d-afdb-4fb9-8400-862a7255dbd7" />

> **Analyse :** Windows utilise encore ce protocole pour les noms simples.  
> Il faut le désactiver manuellement au niveau de la carte réseau car les GPO ne suffisent pas toujours à couper le cache NetBIOS.

### Procédure de désactivation
1. Aller dans les **Propriétés** de la carte réseau > **IPv4** > **Avancé**
2. Onglet **WINS**
3. Cocher : **🔘 Désactiver NetBIOS sur TCP/IP**

<img width="468" height="577" alt="Capture d'écran 2026-01-24 184103" src="https://github.com/user-attachments/assets/f9bd5c2d-7be9-4e4d-8e30-5366485eb228" />

### Nettoyage du cache (indispensable)
NetBIOS garde en mémoire les anciennes requêtes. Il faut purger le cache, sinon l'attaque reste possible temporairement.

* **Condition :** L'invite de commande doit être lancée **en tant qu'administrateur**
* **Commande :**
```cmd
nbtstat -R
<img width="752" height="55" alt="Capture d'écran 2026-01-24 184446" src="https://github.com/user-attachments/assets/94de4be3-1ef5-41c6-9be8-c532e9ac4f2a" />

🏆 Validation Finale (Zero Trust)

J'ai relancé une batterie de tests complète :

Nom long : \\test-long.local

Nom court : \\test

Résultat définitif :

Responder : reste totalement silencieux et vierge.

<img width="863" height="55" alt="Capture d'écran 2026-01-24 184744" src="https://github.com/user-attachments/assets/307e6ed7-a765-4991-81dc-25250f6863fe" />

Client Windows : affiche le même message d'erreur réseau pour les deux cas.

<img width="615" height="188" alt="Capture d'écran 2026-01-24 184727" src="https://github.com/user-attachments/assets/3f5e6fdc-aff9-4987-9c61-5a29cf08db44" />

Conclusion : En fermant successivement LLMNR, mDNS et NetBIOS, j'ai éliminé 100% de la surface d'attaque sur la résolution de noms. Le réseau est durci et l'attaque par empoisonnement est devenue impossible.

## 🚀 Statut
En cours – amélioration continue
