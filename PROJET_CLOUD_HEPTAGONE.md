# Étude de Cas : Infrastructure Cloud "Heptagone"

Ce document détaille la stratégie de déploiement d'un cloud privé pour l'école Heptagone, dimensionné pour **40 étudiants** (160 instances Debian au total).

---

## 1. Analyse du besoin (Workload)

Chaque étudiant doit manipuler une architecture multi-tiers complète pour apprendre les concepts de segmentation et de haute disponibilité.

| Machine | Rôle | Ressources | Justification |
| :--- | :--- | :--- | :--- |
| **Bastion** | Passerelle SSH | 1 vCPU / 512 Mo | Sécurise l'accès au reste de l'infrastructure. |
| **Proxy (Nginx)** | Répartiteur | 1 vCPU / 1 Go | Apprentissage du Load Balancing et du SSL. |
| **Web (x2)** | Serveurs App | 2 vCPU / 4 Go | Simulation de redondance (Apache/PHP/MariaDB). |

**Volume total à supporter :** 160 vCPU / 220 Go RAM / ~3 To de stockage.

### Justification du Stockage
* **Bastion SSH (10 Go)** : Une Debian minimale occupe ~2 Go. 10 Go permettent de stocker confortablement les journaux système et les outils d'administration sans risque de saturation rapide.
* **Nginx Load Balancer (15 Go)** : Dimensionné pour accueillir les logs d'accès HTTP/HTTPS qui peuvent être volumineux et un espace tampon pour le cache des fichiers statiques.
* **Apache/PHP/MariaDB (20 Go)** : Ces instances hébergent à la fois le serveur web, le code applicatif et les données (SGBD). C'est le volume critique pour permettre aux étudiants d'importer des bases de données de taille réaliste.

### Total Infrastructure Cible
* **vCPU** : 160 cœurs (évolutif via overcommit).
* **RAM** : 220 Gio (net).
* **Stockage** : ~2.6 To (utile).

---

## 2. Option A : Le Cloud "On-Premise" (Proxmox VE)

L'idée est d'installer un cluster physique dans la salle serveur de l'école.



### Configuration préconisée
* **Matériel :** 3 serveurs (ex: Dell R640 reconditionnés) avec processeurs AMD EPYC ou Intel Xeon.
* **Mémoire :** 128 Go de RAM par serveur (384 Go au total).
* **Coût estimé (Investissement) :** **~9 700 € HT** (Serveurs, Switchs 10GbE, Onduleur).

### Pourquoi ce choix ?
* **Pédagogie concrète :** Les étudiants peuvent voir les serveurs, comprendre le câblage et la gestion de la chaleur/énergie.
* **Haute Disponibilité ($N+1$) :** Avec 3 nœuds, on peut perdre un serveur entier sans que les TP des étudiants ne s'arrêtent.
* **Indépendance :** Aucun coût mensuel récurrent une fois le matériel acheté.

---

## 3. Option B : Le Cloud "Déporté" (OVHcloud Bare Metal)

Utilisation de serveurs dédiés loués et interconnectés via un réseau privé virtuel (vRack).

* **Configuration :** 3 serveurs de la gamme **Advance-2**.
* **Coût estimé (Fonctionnement) :** **~640 € HT / mois**.

### Pourquoi ce choix ?
* **Zéro maintenance :** L'école ne gère pas les pannes de disques ou l'électricité. C'est idéal si l'équipe technique est réduite.
* **Connectivité :** Les étudiants bénéficient d'une bande passante internet professionnelle (1 Gbps+), cruciale pour télécharger des images Docker ou des paquets.

---

## 4. Alternatives Open Source

Si Proxmox n'est pas retenu, d'autres solutions sont pertinentes pour l'école :

1.  **XCP-ng :** La solution la plus proche des standards industriels (VMware/Citrix). Elle est extrêmement robuste pour la gestion de flottes de VM.
2.  **Harvester :** L'option "moderne". Basé sur Kubernetes, il permet de gérer des VM et des conteneurs sur la même interface. Parfait pour un cursus 100% DevOps.
3.  **OpenNebula :** Très utilisé dans la recherche, il offre un portail "Self-Service" très simple pour les étudiants.

---

## 5. Synthèse et Préconisation

**Le choix Heptagone :** L'option **On-Premise (Proxmox)** est la plus valorisante pédagogiquement. Elle permet d'amortir les coûts en moins de 15 mois et offre une liberté totale sur la configuration réseau. 

Cependant, si la salle serveur de l'école n'est pas climatisée ou sécurisée, l'option **OVHcloud** est préférable pour garantir la continuité des cours.
