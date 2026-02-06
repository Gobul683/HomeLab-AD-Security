## 5. Intégration du Client Windows 10

### 5.1. Configuration DNS
Le point critique : Le client doit impérativement utiliser l'IP du serveur AD (`192.168.10.10`) comme serveur DNS pour résoudre le domaine `lab.lan`.

<img width="655" height="236" alt="Client Network" src="https://github.com/user-attachments/assets/87029898-93e2-4614-bbb2-7c1ce3b4b9f4" />
<img width="455" height="520" alt="Client DNS" src="https://github.com/user-attachments/assets/89f4ec16-651b-4530-832f-d6b13a26b7f7" />

### 5.2. Jonction au Domaine
Connexion réussie au domaine `lab.lan` et authentification avec le compte Administrateur pour autoriser la jonction.

<img width="514" height="368" alt="Domain Join" src="https://github.com/user-attachments/assets/2859cb07-6bfa-4d2d-a315-627111f57c9e" />
<img width="362" height="170" alt="Welcome Msg" src="https://github.com/user-attachments/assets/30520712-68e1-4e27-add9-6840619ef6bf" />

### 5.3. Validation Finale
Ouverture de session avec l'utilisateur `LAB\mugur` et vérification via l'invite de commande (`whoami`, `ipconfig`).

<img width="771" height="617" alt="Final Validation" src="https://github.com/user-attachments/assets/91803fd8-3e14-44a6-9c46-be027c0dbfd4" />
