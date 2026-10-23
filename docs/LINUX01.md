Sur la machine virtuelle LINUX01, 
le serveur Web Nginx a été déployé automatiquement à l’aide du playbook Ansible playbooks/linux.yml. 
Son fonctionnement a ensuite été contrôlé avec la commande systemctl status nginx, qui permet de vérifier que le service est bien actif et qu’il écoute sur le port 80.

Un disque virtuel supplémentaire a également été ajouté afin d’augmenter l’espace de stockage disponible sur la machine. 
Celui-ci a été formaté en ext4, puis configuré pour être accessible depuis le répertoire /mnt/data. 
Pour assurer sa disponibilité après chaque redémarrage, une entrée a été ajoutée dans le fichier /etc/fstab.

Enfin, les journaux du système ont été examinés à l’aide de la commande journalctl. 
La commande journalctl -u nginx permet de consulter les événements associés au serveur Web, tandis que journalctl -e donne accès aux dernières entrées des journaux système
afin de faciliter le suivi de l’activité de la machine et la recherche d’éventuelles erreurs.
