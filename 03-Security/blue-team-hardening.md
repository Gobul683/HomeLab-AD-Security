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
  <img width="1465" height="867" alt="Capture d&#39;écran 2026-01-24 180429" src="https://github.com/user-attachments/assets/329cf28d-bba9-4b71-b128-b9c0091b2595" /> 
  
* **Commande d'application :** `gpupdate /force` (dans une invite de commande).

> [!NOTE]
> **Subtilité technique (Logique GPO) :**
> Le paramètre se nomme *"Désactiver la résolution de noms multidiffusion"*.
> Il faut donc le mettre sur **✅ Activé (Enabled)** pour que l'interdiction prenne effet. (On active la désactivation).
<img width="321" height="145" alt="image" src="https://github.com/user-attachments/assets/a93b1e02-63b2-41ec-8dcd-6c471c0298a5" />


* **Résultat :** Responder ne capture plus de trafic étiqueté `[LLMNR]`.

## 2. La persistance du mDNS (Noms longs)
Lors d'un test avec un nom de domaine complet (`\\test-apres-desactivation.LLMNR`), j'ai remarqué que le protocole **mDNS** prenait le relais. Contrairement au LLMNR, un simple `gpupdate` ne suffit pas toujours à arrêter ce service, un redémarrage est nécessaire.
<img width="1443" height="731" alt="Capture d&#39;écran 2026-01-24 181014" src="https://github.com/user-attachments/assets/513cc2e7-d1fa-426a-95a5-bdb769a1abe5" />

* **Action :** Modification de la GPO pour cibler spécifiquement *Configurer le protocole DNS de multidiffusion*.

* **Chemin GPO :** Identique au précédent (toujours dans le dossier *Client DNS* de la *Default Domain Policy*).
* **Paramètre :** *Configurer le protocole DNS de multidiffusion (mDNS)*.
<img width="1460" height="858" alt="Capture d&#39;écran 2026-01-24 182049" src="https://github.com/user-attachments/assets/1b8e332a-488d-45aa-851e-de22eb348c4f" />
* **Valeur :** 🚫 **Désactivé**.

> [!WARNING]
> **Attention à la logique inversée :**
> Contrairement au paramètre LLMNR précédent, ici l'intitulé est *"Configurer..."*.
> Nous choisissons donc **Désactivé** pour dire "Je ne veux pas de cette configuration". (On désactive la fonctionnalité).
<img width="400" height="145" alt="image" src="https://github.com/user-attachments/assets/97aa7171-ed23-4603-8473-903847b2ae1f" />

* **Application :** **Redémarrage obligatoire** du poste client pour purger le service *Dnscache*.

**Test intermédiaire :**
En tentant d'accéder à `\\test-apres-configuration-mDNS`, Windows m'affiche désormais un message d'erreur réseau immédiat. C'est le comportement attendu.
<img width="247" height="34" alt="Capture d&#39;écran 2026-01-24 182501" src="https://github.com/user-attachments/assets/790a785e-2c21-48f6-9492-5e82e662ea12" />
<img width="609" height="183" alt="Capture d&#39;écran 2026-01-24 183230" src="https://github.com/user-attachments/assets/c4d05889-f655-4f6a-83f8-ffbce4993ab5" />

## 3. La surprise du NetBIOS (Noms courts)
Pensant être sécurisé, j'ai réalisé un dernier test avec un nom court : `\\test`.
**Surprise :** Responder s'est réveillé et a capturé des hashs via le protocole **NBT-NS** (NetBIOS) !
<img width="1438" height="691" alt="Capture d&#39;écran 2026-01-24 183407" src="https://github.com/user-attachments/assets/704e2a5d-afdb-4fb9-8400-862a7255dbd7" />


*(Insère ici ton screen de Responder qui s'active avec le tag [NBT-NS])*

> **Analyse :** Windows utilise encore ce vieux protocole pour les noms simples. Il faut impérativement le tuer manuellement au niveau de la carte réseau car les GPO ne suffisent pas toujours à couper le cache NetBIOS.

### Procédure de désactivation :
1.  Aller dans les **Propriétés** de la carte réseau > **IPv4** > **Avancé**.
2.  Onglet **WINS**.
3.  Cocher : **🔘 Désactiver NetBIOS sur TCP/IP**.
<img width="468" height="577" alt="Capture d&#39;écran 2026-01-24 184103" src="https://github.com/user-attachments/assets/f9bd5c2d-7be9-4e4d-8e30-5366485eb228" />

### Nettoyage du Cache (Indispensable)
NetBIOS garde en mémoire les anciennes requêtes. Il faut purger le cache, sinon l'attaque reste possible temporairement.
* **Condition :** L'invite de commande doit être lancée **"En tant qu'administrateur"**.
* **Commande :**
  ```cmd
  nbtstat -R
*<img width="752" height="55" alt="Capture d&#39;écran 2026-01-24 184446" src="https://github.com/user-attachments/assets/94de4be3-1ef5-41f6-9be8-c532e9ac4f2a" />

## 🏆 Validation Finale (Zero Trust)
J'ai relancé une batterie de tests complète :
1.  Nom long : `\\test-long.local`
2.  Nom court : `\\test`

**Résultat Définitif :**
* **Responder :** Reste totalement silencieux et vierge. Plus aucune ligne ne s'affiche, peu importe le nom demandé.
*<img width="863" height="55" alt="Capture d&#39;écran 2026-01-24 184744" src="https://github.com/user-attachments/assets/307e6ed7-a765-4991-81dc-25250f6863fe" />*
* **Client Windows :** Affiche le même message d'erreur réseau pour les deux cas (preuve qu'il ne trouve personne à qui parler).
*<img width="615" height="188" alt="Capture d&#39;écran 2026-01-24 184727" src="https://github.com/user-attachments/assets/3f5e6fdc-aff9-4987-9c61-5a29cf08db44" />*


> **Conclusion :** En fermant successivement LLMNR, mDNS et NetBIOS, j'ai éliminé 100% de la surface d'attaque sur la résolution de noms. Le réseau est durci et l'attaque par empoisonnement est devenue impossible.
