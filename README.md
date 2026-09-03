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
