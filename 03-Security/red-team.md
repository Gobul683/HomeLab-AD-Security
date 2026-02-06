# ⚔️ Phase 2 : Sécurité Offensive (Red Teaming)

Maintenant que l'infrastructure est fonctionnelle, j'ai basculé en mode "Attaquant" (depuis la machine Kali Linux dans la zone USERS) pour auditer la sécurité de la configuration par défaut de l'Active Directory.

## 1. Reconnaissance & Cartographie
L'objectif est d'identifier les machines vivantes et les services exposés sans connaitre l'architecture au préalable.

* **Outil utilisé :** `Nmap`
* **Commande :** `nmap -sV -O 192.168.10.10`
* **Résultats :**
    * Découverte des ports critiques ouverts : **88** (Kerberos), **389** (LDAP), **445** (SMB).
    * Confirmation qu'il s'agit d'un Contrôleur de Domaine Windows Server.

<img width="736" height="664" alt="Capture d&#39;écran 2026-01-24 164232" src="https://github.com/user-attachments/assets/59f836b7-528b-4bef-a09b-286cf9a13f00" />

## 2. Attaque Man-in-the-Middle (LLMNR Poisoning)
Windows utilise par défaut le protocole **LLMNR** (Link-Local Multicast Name Resolution) pour chercher des noms de machines sur le réseau local s'il ne les trouve pas via le DNS.

* **Scénario :** Un utilisateur fait une erreur de frappe en cherchant un dossier partagé (ex: `\\serveur-inconnu`).
* **L'attaque :**
    * L'outil **Responder** écoute le réseau.
    * Lorsqu'il entend la requête LLMNR, il répond "C'est moi le serveur !".
    * La victime envoie alors son **Hash NTLMv2** (son mot de passe chiffré) à l'attaquant.

<img width="1453" height="798" alt="Capture d&#39;écran 2026-01-24 165045" src="https://github.com/user-attachments/assets/ac1bfff9-40a3-4b13-a897-7746a6811c2d" />

## 3. Cassage de Mot de Passe (Cracking)
Une fois le hash NTLMv2 récupéré, il est inexploitable tel quel pour se connecter. Il faut le "casser" pour retrouver le mot de passe en clair.

* **Outil utilisé :** `John the Ripper`
* **Méthode :** Attaque par dictionnaire (Wordlist).
* **Résultat :** Le mot de passe de l'utilisateur `mugur` a été retrouvé.

> **Preuve de concept :**
> `mugur:rocknroll25!` (Le mot de passe a été cracké avec succès).

*<img width="880" height="310" alt="Capture d&#39;écran 2026-01-24 170737" src="https://github.com/user-attachments/assets/ebf337cc-0eeb-440e-beba-547ec733c645" />*
