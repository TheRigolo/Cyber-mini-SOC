## Installation de Wazuh Manager 4.14

**Environnement** : VM Ubuntu 22.04.5 LTS, IP 192.168.56.10/24, réseau Host-Only.

**Étapes réalisées** :
- Provisionnement de la VM (VirtualBox, host-only)
- Installation du stack Wazuh 4.14 (Manager + Indexer + Dashboard) via le script d'installation officiel

## Problème : espace disque insuffisant lors de l'installation de Wazuh

**Symptôme** : l'installation du stack Wazuh (Manager + Indexer + Dashboard) 
a échoué en cours de décompression/installation, avec un message lié à un 
espace disque insuffisant.

**Diagnostic** : le `df -h` lancé juste après l'échec affichait 41% 
d'utilisation (14G disponibles sur 24G), ce qui semblait largement 
suffisant. L'explication la plus probable : l'installation (dpkg/apt) 
fonctionne de manière transactionnelle et a fait un rollback après 
l'échec, ce qui a libéré l'espace qui était temporairement occupé pendant 
la décompression du Dashboard et d'OpenSearch, deux composants volumineux. 
Le disque a donc probablement été saturé pendant quelques instants au 
moment de l'installation, sans que ça se voie une fois l'échec passé.

**Résolution** :
1. Redimensionnement du disque virtuel via VirtualBox
2. Extension du volume logique : `lvextend` + `resize2fs`
3. Relance de l'installation → succès

**Ce que j'en retiens** :
- Le stack Wazuh (OpenSearch en particulier) demande beaucoup d'espace 
  disque, même de façon temporaire pendant l'installation, mieux vaut 
  prévoir large dès le départ (50 Go recommandés).
- Un `df -h` lancé après coup peut induire en erreur si l'outil 
  d'installation fait un rollback automatique. Pour un vrai diagnostic, 
  il aurait fallu surveiller l'espace disque en direct pendant 
  l'installation (`watch -n 1 df -h`) ou regarder les logs d'installation.




**Résultat** : dashboard accessible, aucun agent connecté à ce stade (attendu).
