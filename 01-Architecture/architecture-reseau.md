---

## 2. Architecture Réseau & Segmentation

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

---
