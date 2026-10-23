# Rapport d'incident

## 1. Symptôme observé

Le playbook `verification.yml` échoue sur la tâche
« Vérifier la résolution DNS du contrôleur de domaine » :
`getent hosts WIN-DC01.infraready.test` renvoie un code de retour 2
(nom non résolu).

## 2. Tests et journaux consultés

| Test | Commande | Résultat |
|------|----------|----------|
| Résolution via le DC | `nslookup WIN-DC01.infraready.test 192.168.1.200` | OK (192.168.1.200) |
| DNS utilisé par LINUX01 | `resolvectl status` | Current DNS Server : 8.8.8.8 |
| Configuration réseau | `cat /etc/netplan/50-cloud-init.yaml` | Deux DNS : 192.168.1.200 et 8.8.8.8 |
| Résolution par défaut | `getent hosts WIN-DC01.infraready.test` | Échec (rc=2) |

## 3. Cause identifiée

Le fichier Netplan de LINUX01 contenait deux serveurs DNS :
192.168.1.200 (DC) et 8.8.8.8 (Google).
`systemd-resolved` a sélectionné 8.8.8.8 comme serveur courant.
Ce DNS public ne connaît pas la zone `infraready.test`,
donc la résolution du domaine interne échouait.

## 4. Correction appliquée

1. Édition de `/etc/netplan/50-cloud-init.yaml` :
   suppression de `8.8.8.8`, conservation de `192.168.1.200` seul,
   ajout de `search: [infraready.test]`.
2. Application : `sudo netplan apply`.
3. Ajout d'un redirecteur DNS sur WIN-DC01 pour conserver
   la résolution Internet : `Add-DnsServerForwarder -IPAddress 8.8.8.8`.

## 5. Vérification du retour à la normale

| Test | Commande | Résultat |
|------|----------|----------|
| DNS courant | `resolvectl status` | Current DNS Server : 192.168.1.200 |
| Résolution interne | `getent hosts WIN-DC01.infraready.test` | 192.168.1.200 |
| Résolution Internet | `sudo apt update` | OK |
| Playbook de vérification | `ansible-playbook verification.yml` | LINUX01 : failed=0 |
