# Proposition de projet

À réaliser en binôme.

## Heptagone

L'école Heptagone souhaite monter un cloud _on premise_ pour héberger l'infrastrucuture
de certains cours DevOps et Cloud (et autres).

L'idée a été proposé d'installer un cluster Proxmox sur des machines physiques dans nos locaux

Évaluer les coûts si on souhaite que deux classes de 20 étudiants puisse en même 
temps déployer, pour chaque étudiant.e quatre instances Debian GNU/Linux :
- Un bastion avec 512Mio de RAM et un seul CPU
- Une instance faisant tourner NGINX comme mandataire inverse _(équilibrage de charge)_
- Deux instances faisant tourner un serveur Web Apache avec PHP et un serveur MariaDB

Évaluer les coûts si au lieu de machines dans les locaux on louait des serveurs physique
dédiés chez OVH.

Format : un document Markdown dans un dépôt GIT, deux pages max 
si on l'imprimait.

BONUS :

- Il y a d'autre "stacks" de Cloud Open Source que Proxmox :
  - Open Stack
  - Trouvez en d'autres

Lesquelles pourraient être considérées par l'école.


