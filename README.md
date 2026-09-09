# Projet Mini SOC (Security Operations Center)

Ce projet a pour objectif de concevoir et de déployer un **mini SOC** local virtualisé afin de mettre en œuvre des mécanismes de détection des menaces, de centralisation des logs et d'analyse de sécurité (Blue Team vs Red Team).


## Architecture du Réseau

L'ensemble du laboratoire est hébergé sur un hyperviseur (VirtualBox) et utilise un **réseau isolé de type Host-Only** (`192.168.56.0/24`) pour garantir la sécurité et l'étanchéité de l'environnement face au réseau réel.

```text
 _________________________________________________________________
|                          HYPERVISEUR (hôte)                     |
|                                                                 |
|                                                                 |
|   ┌───────────────┐     ┌───────────────┐     ┌───────────────┐ │
|   |  VM Wazuh     | <-- |  VM Ubuntu    |     |  VM Windows   | │
|   |  Manager +    |     |  (agent +     |     |  (agent +     | │
|   |  Indexer +    |     |   cible)      |     |   cible)      | │
|   |  Dashboard    |     └───────────────┘     └───────────────┘ │
|   |  + Suricata   |              ▲                     ▲        |
|   └───────────────┘              │                     │        |
|           ▲                      └────── logs/alertes ─┘        |
|           │                                                     |
|   ┌───────────────┐                                             |
|   |  VM Kali      |                                             |
|   |  (attaque)    |                                             |
|   └───────────────┘                                             |
|_________________________________________________________________|

```

## Composants de l'architecture Wazuh

- **Wazuh Indexer** : stocke et indexe toutes les données collectées 
  (logs, alertes). Basé sur OpenSearch, c'est lui qui permet la recherche 
  et l'agrégation des données dans le Dashboard.
- **Wazuh Manager** : reçoit les événements envoyés par les agents, les 
  compare à des règles de détection (decoders + rules) et génère les 
  alertes lorsqu'un comportement suspect est identifié.
- **Wazuh Dashboard** : interface web permettant de visualiser les 
  alertes, l'état des agents et les tableaux de bord (basé sur 
  OpenSearch Dashboards).
- **Wazuh Agent** : programme léger installé sur chaque machine 
  surveillée (Linux, Windows...). Il collecte les logs et événements 
  locaux et les transmet au Manager.

Dans ce lab, le Manager, l'Indexer et le Dashboard sont installés sur la 
même VM (`wazuh-manager`, 192.168.56.10) — une architecture "all-in-one" 
adaptée à un environnement de lab, contrairement à un vrai déploiement 
en production où ces composants seraient généralement séparés sur 
plusieurs serveurs pour la charge et la résilience.




### Adressage IP

| Machine | Rôle | IP |
|---|---|---|
| wazuh-manager | Manager + Indexer + Dashboard + Suricata | 192.168.56.10/24 |
| target-linux | Agent Wazuh | 192.168.56.20/24 |
| target-windows | Agent Wazuh | 192.168.56.30/24 |
| attacker-kali | Machine d'attaque | 192.168.56.40/24 |



## Stack utilisée

| Composant | Version |
|---|---|
| Hyperviseur | VirtualBox |
| SIEM | Wazuh 4.14 (Manager + Indexer + Dashboard) |
| OS cible Linux | Ubuntu Server 22.04.5 LTS |
| OS cible Windows | Windows 11 |
| Attaque | Kali Linux |
| Réseau | Host-Only, 192.168.56.0/24 |
| IDS réseau | Suricata 6.0.4 (dépôt Ubuntu par défaut) |
