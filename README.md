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
### Adressage IP

| Machine | Rôle | IP |
|---|---|---|
| wazuh-manager | Manager + Indexer + Dashboard + Suricata | 192.168.56.10/24 |
| target-linux | Agent Wazuh | 192.168.56.x/24 |
| target-windows | Agent Wazuh | 192.168.56.x/24 |
| attacker-kali | Machine d'attaque | 192.168.56.x/24 |



## Stack utilisée

| Composant | Version |
|---|---|
| Hyperviseur | VirtualBox |
| SIEM | Wazuh 4.14 (Manager + Indexer + Dashboard) |
| OS cible Linux | Ubuntu Server 22.04.5 LTS |
| OS cible Windows | Windows 11 |
| Attaque | Kali Linux |
| Réseau | Host-Only, 192.168.56.0/24 |
