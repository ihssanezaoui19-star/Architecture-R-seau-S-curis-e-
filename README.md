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

## Faiblesses identifiées lors des tests

Ces problèmes ont été découverts pendant la phase de tests et corrigés. Ils sont mentionnés ici par transparence — faire des erreurs et les corriger fait partie de l'apprentissage.

**T-03 — Accès WebGUI OPNsense non restreint**
La page de login OPNsense était accessible depuis le VLAN USERS. La règle de restriction n'était pas correctement appliquée. Correction : règle firewall ajoutée pour bloquer l'accès à l'interface admin hors VLAN ADMIN.

**T-07 — Traçabilité Wazuh incomplète**
Wazuh recevait les logs Squid mais n'extrayait pas le nom d'utilisateur, seulement l'IP. Correction : modification du décodeur Wazuh pour parser correctement le champ username.

**T-09 — Clients VPN avec accès non restreint**
Par défaut, OpenVPN appliquait une règle "allow all traffic" permettant aux clients VPN d'accéder à tous les VLANs, y compris ADMIN et WebGUI OPNsense. Correction : suppression de la règle automatique et création de règles explicites limitant les clients VPN aux ressources autorisées uniquement.

**T-11 — Détection Nmap partielle**
Suricata ne détectait pas les scans Nmap à cause de règles obsolètes. Correction : mise à jour des rulesets Emerging Threats et ajout de sources supplémentaires (Abuse.ch).

**T-12 — Détection Metasploit insuffisante**
Les signatures Suricata ne couvraient pas le trafic Metasploit sur le port 4444. Correction : mise à jour des rulesets avec signatures C2 et Metasploit.

---

## Note importante

Ce document représente l'état de l'infrastructure à un moment donné. Il est possible que vous constatiez des modifications, des ajustements ou des faiblesses qui ne sont pas documentés ici.

Chaque jour et chaque nouvelle chose que j'apprends me permet de revoir mes choix, d'identifier des faiblesses et d'améliorer l'architecture. Je suis encore en apprentissage, ce projet n'est donc pas terminé — il y a encore beaucoup de choses à ajouter, corriger et explorer.

Si vous avez des remarques, des conseils ou des suggestions, n'hésitez pas à me contacter :

📧 ihssanezaoui19@gmail.com

## Auteur

**ZAOUI Ihssane** — Projet personnel cybersécurité · 2026
