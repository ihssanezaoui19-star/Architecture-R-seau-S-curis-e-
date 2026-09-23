# Architecture réseau segmentée et supervisée

**Projet personnel de laboratoire sur VMware Workstation (2026), en évolution.** Le rapport utilise une organisation fictive et un plan d'adressage de test. Cette présentation se concentre sur les choix techniques et les vérifications utiles à un poste junior en sécurité réseau et Blue Team.

## Problématique

Comment empêcher qu'un poste utilisateur, un invité ou un client VPN atteigne librement les services d'administration, tout en gardant un accès Web contrôlé et des traces exploitables en cas d'incident ? Dans un réseau plat, une règle trop large au pare-feu ou au VPN peut annuler les protections prévues par le dessin de l'architecture.

## Architecture et travail réalisé

| Couche | Configuration documentée | Vérification ou point corrigé |
| --- | --- | --- |
| Segmentation | Zones utilisateurs, administration, invités, DMZ et accès distant dans OPNsense | Revue des règles inter-zones ; accès à la console d'administration restreint |
| Sortie Web | Proxy Squid, ACL de filtrage et journalisation | Capture d'un refus d'accès à une destination bloquée |
| Accès distant | OpenVPN avec certificats et MFA TOTP | Règle trop large des clients VPN identifiée puis remplacée par des accès explicites (README du projet) |
| Détection | Suricata en **mode miroir** et collecte centralisée dans Wazuh, avec Sysmon sur Windows | Traces utiles aux investigations ; ajustements de règles et décodage du nom d'utilisateur mentionnés dans le README |

![Extrait recadré d'un accès Web refusé par les ACL Squid dans la maquette](images/capture-proxy-acces-refuse.png)

## Résultats observés et limites

Le rapport montre les paramètres et des vérifications ponctuelles, notamment une navigation autorisée et un refus par ACL du proxy. Le README d'origine consignait aussi cinq problèmes rencontrés en test (console OPNsense accessible depuis la mauvaise zone, règle VPN trop large, journal Wazuh incomplet et signatures Suricata à mettre à jour) ainsi que les corrections apportées ; ces cas servent ici à expliquer la méthode de diagnostic. Le document seul ne démontre ni « couverture totale » des attaques ni un score Zero Trust universel.

**Suricata analyse ici une copie du trafic : il alerte, mais ne bloque pas lui-même les paquets.** Un titre « mini SOAR » figure à la fin du rapport sans procédure ni preuve d'exécution ; je ne présente donc pas un blocage automatique comme résultat livré.

## Documentation

- [Méthodologie pas à pas : règles, proxy, VPN, détection et corrections](METHODOLOGIE.md)
- [Schéma simplifié sans plan d'adressage](images/architecture-anonymisee.svg)

Le rapport technique déjà présent dans ce dépôt porte la mention « Confidentiel — Usage interne uniquement » et contient des détails de laboratoire. Les visuels de cette page sont recadrés ou reconstruits sans nom d'organisation, adresse, compte ou secret. Sa présence dans l'historique GitHub demande une revue distincte de confidentialité.
