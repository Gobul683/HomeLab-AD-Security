## 4. Mise en place de l'Active Directory

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
