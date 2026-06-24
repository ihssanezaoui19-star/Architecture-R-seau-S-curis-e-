# Architecture Réseau Sécurisée — VMware Workstation

Projet personnel de cybersécurité réseau simulant une infrastructure d'entreprise complète sur **VMware Workstation**, basée sur le modèle **Zero Trust**.

## Technologies déployées

| Composant | Technologie | Rôle |
|-----------|-------------|------|
| Firewall | OPNsense | Firewall stateful, NAT, VPN, règles inter-VLAN |
| Proxy web | Squid | Filtrage web, SSL Bump, authentification, cache |
| IDS/IPS | Suricata | Détection d'intrusions en mode mirroring |
| VPN | OpenVPN | Accès distant avec certificats X.509 + MFA TOTP |
| SIEM | Wazuh | Centralisation des logs, corrélation, alertes |
| Endpoint | Sysmon | Surveillance activité Windows en temps réel |
| Mini SOAR | Scripts Wazuh + API OPNsense | Réponse automatique aux incidents |

## Ce que couvre ce projet

- Configuration complète du firewall OPNsense (règles, NAT, DNS, NTP, backup)
- Proxy Squid avec SSL Inspection, ACL, blocage malware et réseaux sociaux
- Segmentation en VLANs isolés par rôle (Admin, Users, Guests, DMZ, Servers)
- VPN OpenVPN avec certificats et authentification MFA
- Déploiement complet de Wazuh (Indexer, Manager, Dashboard, Agents)
- Règles Wazuh personnalisées pour OPNsense, Squid et Suricata
- Mini SOAR : blocage automatique via l'API OPNsense sur alerte Wazuh

## Document

Le guide technique complet est disponible dans ce dépôt :
`Architecture_Réseau_Sécurisée.docx`

## Note importante

Ce document représente l'état de l'infrastructure à un moment donné. Il est possible que vous constatiez des modifications, des ajustements ou des faiblesses qui ne sont pas documentés ici.

Chaque jour et chaque nouvelle chose que j'apprends me permet de revoir mes choix, d'identifier des faiblesses et d'améliorer l'architecture. Je suis encore en apprentissage, ce projet n'est donc pas terminé — il y a encore beaucoup de choses à ajouter, corriger et explorer.

Si vous avez des remarques, des conseils ou des suggestions, n'hésitez pas à me contacter :

📧 ihssanezaoui19@gmail.com

## Auteur

**ihssane zaoui** — Projet personnel cybersécurité · 2026
