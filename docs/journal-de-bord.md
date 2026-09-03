## Problème : espace disque insuffisant lors de l'installation de Wazuh

**Symptôme** : l'installation d'OpenSearch/Wazuh Indexer échouait avec [erreur exacte].

**Diagnostic** : `df -h` a montré que la partition root était pleine à 41%.
Cause : disque virtuel alloué trop petit au départ (20 Go au lieu de 50 Go recommandés).

**Résolution** :
1. Redimensionnement du disque virtuel via VirtualBox
2. Extension de la partition avec `lvextend` + `resize2fs`
3. Relance de l'installation

**Ce que j'en retiens** : toujours prévoir large sur le stockage pour un stack
ELK/OpenSearch, qui indexe beaucoup de données même en lab.
