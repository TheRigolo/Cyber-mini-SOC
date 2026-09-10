# Règles de détection personnalisées — Partie 6

## Tableau de couverture MITRE ATT&CK

| Rule ID | Description | Technique MITRE | Niveau | Testée | Résultat |
|---|---|---|---|---|---|
| 100010 | Brute force SSH (4 échecs/2min) | T1110 - Brute Force | 10 | ✅ | Alerte déclenchée après 4 tentatives depuis Kali |
| 100011 | Création d'utilisateur | T1136.001 - Create Account | 12 | ✅ | Alerte de niveau 12 déclenchée suite au `useradd` |
| 100012 | Accès à /etc/shadow via sudo | T1003.008 - OS Credential Dumping | 9 | ✅ | Alerte déclenchée instantanément via l'analyse du log d'authentification |
| 100013 | Modification fichier critique (FIM) | T1565.001 - Data Manipulation | 12 | ✅ | Alerte déclenchée immédiatement grâce au paramétrage temps réel (`realtime="yes"`) |

## Détail par règle

### 100010 — Brute force SSH
**Règle XML** : 
```xml
<rule id="100010" level="10" frequency="4" timeframe="120">
  <if_matched_sid>5760</if_matched_sid>
  <same_source_ip />
  <description>Brute force SSH détecté - 4 échecs en 2 minutes depuis la même IP</description>
  <mitre>
    <id>T1110</id>
  </mitre>
  <group>authentication_failures,pci_dss_10.2.4,pci_dss_10.2.5,</group>
</rule>
```

**Test réalisé** : Exécution de 4 tentatives consécutives de connexion SSH avec des identifiants erronés depuis la machine d'attaque Kali vers l'agent Linux, le tout en moins de 2 minutes.

**Résultat** :
![Brute_force_ssh](Screenshots/Rule_100010.png)

Sur cette capture, on observe clairement le déclenchement de la règle personnalisée 100010 avec un niveau de sévérité critique de 10. Cela confirme que le moteur Wazuh a correctement corrélé les
multiples événements d'échec (règle parente 5760) grâce aux attributs `frequency="4"` et `timeframe="120"` que nous avons intégrés à la balise principale.


### 100011 — Création d'utilisateur
**Règle XML** :
```
<rule id="100011" level="12">
  <if_sid>5902</if_sid>
  <description>Création d'un nouvel utilisateur détectée sur le système</description>
  <mitre>
    <id>T1136.001</id>
  </mitre>
  <group>account_creation,</group>
</rule>
```

**Test réalisé** : Exécution de la commande `sudo useradd testuser` directement sur le terminal de l'agent Linux.

**Résultat** :
![Creation_utilisateur](Screenshots/Rule_100011.png)

La capture montre que l'action a bien été interceptée et catégorisée sous notre ID 100011 (niveau 12), écrasant la règle par défaut de niveau 8. Pour obtenir ce résultat, 
un ajustement a été nécessaire : la condition `<match>useradd|adduser</match>`a été supprimée car elle était trop stricte par rapport au log brut (full_log) envoyé par le système.



### 100012 — Accès à /etc/shadow via sudo
**Règle XML** :
```
<rule id="100012" level="9">
  <if_sid>5402</if_sid>
  <match>/etc/shadow|/etc/passwd</match>
  <description>Tentative d'accès à un fichier sensible via sudo</description>
  <mitre>
    <id>T1003.008</id>
  </mitre>
  <group>privilege_escalation,</group>
</rule>
```

**Test réalisé** : Exécution de la commande `sudo cat /etc/shadow` sur l'agent Linux.

**Résultat** : 
![Acces_doc_sudo](Screenshots/Rule_100012.png)

Comme illustré, la règle 100012 de niveau 9 est remontée instantanément. L'analyseur de logs a parfaitement détecté en temps réel l'utilisation de la commande sudo (déclenchant la règle 5402) 
contenant la chaîne de caractères `/etc/shadow` définie dans notre balise `<match>`.



### 100013 — Modification fichier critique (FIM)
**Règle XML** :
```
<rule id="100013" level="12">
  <if_sid>550,553,554</if_sid>
  <match>/etc/passwd|/etc/shadow|/etc/sudoers</match>
  <description>Modification d'un fichier système critique détectée (FIM)</description>
  <mitre>
    <id>T1565.001</id>
  </mitre>
  <group>syscheck,</group>
</rule>
```

**Test réalisé** : Ajout d'une ligne de commentaire factice à la fin du fichier `/etc/passwd` en utilisant l'éditeur `nano` sur l'agent Linux.

**Résultat** :
![Modif_fichier_critique](Screenshots/Rules_100013.png)

La capture confirme le déclenchement immédiat de l'alerte d'intégrité (ID 100013). Pour que cette alerte remonte instantanément, il a fallu configurer l'agent (`ossec.conf`) 
en ajoutant `<directories realtime="yes" report_changes="yes">/etc/passwd</directories>` pour résoudre un conflit de configuration qui l'écrasait.
