# Étude de Faisabilité : Cloud IaaS Heptagone

Ce document présente l'analyse technique et financière pour la mise en place d'une infrastructure de virtualisation dédiée aux cours DevOps et Cloud.

## Analyse du Dimensionnement (Workload)

L'objectif est de supporter **40 étudiants** simultanément (2 classes de 20). Chaque étudiant dispose de 4 instances Debian, soit un total de **160 machines virtuelles (VM)**.

### Détail des ressources par étudiant
| Instance | Rôle | vCPU | RAM | Stockage (estimé) |
| :--- | :--- | :--- | :--- | :--- |
| **VM 1** | Bastion SSH | 1 | 512 Mio | 10 Go |
| **VM 2** | Nginx (Load Balancer) | 1 | 1 Gio | 15 Go |
| **VM 3** | Apache/PHP/MariaDB (Srv A) | 1 | 2 Gio | 20 Go |
| **VM 4** | Apache/PHP/MariaDB (Srv B) | 1 | 2 Gio | 20 Go |
| **Total/Étudiant** | - | **4 vCPU** | **5.5 Gio** | **65 Go** |

### Justification du Stockage
* **Bastion SSH (10 Go)** : Une Debian minimale occupe ~2 Go. 10 Go permettent de stocker confortablement les journaux système et les outils d'administration sans risque de saturation rapide.
* **Nginx Load Balancer (15 Go)** : Dimensionné pour accueillir les logs d'accès HTTP/HTTPS qui peuvent être volumineux et un espace tampon pour le cache des fichiers statiques.
* **Apache/PHP/MariaDB (20 Go)** : Ces instances hébergent à la fois le serveur web, le code applicatif et les données (SGBD). C'est le volume critique pour permettre aux étudiants d'importer des bases de données de taille réaliste.

### Total Infrastructure Cible
* **vCPU** : 160 cœurs (évolutif via overcommit).
* **RAM** : 220 Gio (net).
* **Stockage** : ~2.6 To (utile).

---

## Cluster On-Premise (Proxmox VE)

Installation de machines physiques dans les locaux de l'école. Pour garantir la disponibilité, nous préconisons un cluster de **3 nœuds**.

### Configuration Matérielle (par nœud)
* **Processeur** : 1x AMD EPYC 7313P (16 Cores / 32 Threads).
* **Mémoire** : 128 Gio DDR4 ECC (Total cluster : 384 Gio).
    * *Note : Cela permet de perdre 1 nœud sans interrompre le service (Haute Disponibilité).*
* **Stockage** : 2x 1.92 To NVMe (ZFS Mirror) pour l'OS et les VM.
* **Réseau** : Double interface 10GbE pour le stockage distribué (Ceph) et la migration à chaud.

### Estimation des Coûts (CAPEX)
| Poste | Détails | Prix HT (estimé) |
| :--- | :--- | :--- |
| Serveurs (x3) | Reconditionnés récents (Dell R640 ou équivalent) | 7 500 € |
| Switchs & Câblage | 1x Switch 10GbE SFP+ + 1x Switch GbE | 1 000 € |
| Onduleur (UPS) | 3kVA rackable | 1 200 € |
| **TOTAL** | **Investissement Initial** | **9 700 €** |

---

## Serveurs Dédiés OVHcloud (Bare Metal)

Location de serveurs physiques avec réseau privé (vRack) pour interconnecter les nœuds Proxmox.

### Modèle retenu : Advance-2 (x3)
* **CPU** : Intel Xeon-E 2388G (8c/16t).
* **RAM** : 128 Gio.
* **Disques** : 2x 960 Go SSD.

### Estimation des Coûts (OPEX)
| Poste | Mensualité HT | Annuité HT |
| :--- | :--- | :--- |
| Location 3 serveurs | ~540 € | 6 480 € |
| Options (IP Failover / vRack) | ~100 € | 1 200 € |
| **TOTAL** | **~640 € / mois** | **7 680 € / an** |

> **Comparaison rapide** : Le point d'équilibre (ROI) de l'option On-Premise se situe à environ **15 mois** d'utilisation par rapport à la location chez OVH.

---

## Alternatives Cloud Open Source

En dehors de Proxmox et d'OpenStack (souvent jugé trop complexe pour une petite structure), voici trois alternatives pertinentes :

1.  **XCP-ng (Xen Cloud Platform)** :
    * *Description* : Basé sur l'hyperviseur Xen. Couplé à **Xen Orchestra**, il offre une gestion web très complète.
    * *Pourquoi Heptagone ?* Très stable, sauvegarde native performante, et moins "bricolage" que Proxmox pour certains administrateurs.

2.  **OpenNebula** :
    * *Description* : Une solution de Cloud simple et légère qui gère KVM, LXC et même VMware.
    * *Pourquoi Heptagone ?* Plus proche d'une expérience "AWS-like" pour les étudiants, avec un portail utilisateur simplifié.

3.  **Harvester (by SUSE/Rancher)** :
    * *Description* : Infrastructure hyper-convergée (HCI) moderne basée sur Kubernetes et KubeVirt.
    * *Pourquoi Heptagone ?* Permet de gérer des VM et des conteneurs sur la même couche. Idéal pour un cursus 100% DevOps moderne.

---

## Recommandation Finale

Pour l'école Heptagone, le choix de **Proxmox sur serveurs On-Premise** est recommandé si l'école dispose d'une salle serveur climatisée et d'une bonne connexion fibre. Cela permet aux étudiants de manipuler physiquement la couche réseau. Si la maintenance matérielle est une contrainte, la solution **OVHcloud** est la plus sécurisante.
