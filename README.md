# Architecture réseau segmentée — démarche Zero Trust

**Laboratoire personnel sous VMware Workstation (2026).** J'ai simulé un réseau d'entreprise pour limiter les accès entre zones et réunir les traces utiles à une investigation. « Zero Trust » décrit la démarche de vérification et de limitation des accès ; ce projet n'est pas une certification de sécurité.

## But du projet

Permettre aux utilisateurs, administrateurs, invités et clients VPN d'accéder **uniquement aux ressources nécessaires**, tout en conservant des journaux exploitables lorsqu'un accès est refusé ou suspect. Le laboratoire combine OPNsense, Squid, OpenVPN avec MFA, Suricata et Wazuh.

## Problématique

Un réseau peut paraître segmenté sur un schéma, mais une règle de pare-feu ou de VPN trop large ouvre encore l'administration à d'autres zones. Comment vérifier les droits réels, contrôler la sortie Web et voir une activité anormale sans casser les usages légitimes ?

## Ce que j'ai fait et pourquoi

| Action documentée | Pourquoi | Contrôle ou observation |
| --- | --- | --- |
| Séparer utilisateurs, administration, invités, serveurs et DMZ derrière OPNsense | Réduire la circulation libre entre zones | Règles inter-zones revues ; accès WebGUI trop large identifié puis restreint |
| Déployer Squid avec ACL et journalisation | Contrôler les sorties Web et garder une trace des requêtes | Navigation autorisée et refus ciblé montrés dans le rapport |
| Configurer OpenVPN avec certificats et MFA | Donner un accès distant vérifié et limité | Règle initiale « allow all » retirée au profit d'autorisations explicites |
| Envoyer journaux et événements vers Wazuh, avec Sysmon sur Windows | Rapprocher activité réseau et postes pour l'analyse | Décodeur Squid ajusté afin de récupérer le nom d'utilisateur |
| Configurer Suricata sur une copie du trafic | Détecter certaines activités sans l'insérer dans le chemin réseau | Sources de règles mises à jour après une détection initialement incomplète |

![Schéma simplifié sans noms ni plan d'adressage](images/architecture-anonymisee.svg)

## Résultats observés

Le rapport montre des réglages de segmentation, VPN, proxy et supervision, ainsi qu'un **refus Web effectif par ACL Squid**. Le README initial consignait des erreurs trouvées pendant les essais puis des corrections : WebGUI visible depuis la mauvaise zone, VPN trop permissif, identité absente de certains journaux et signatures Suricata insuffisantes. Les captures disponibles attestent des configurations et de certains essais ; elles ne constituent pas une campagne exhaustive de validation.

![Extrait recadré d'un accès Web refusé](images/capture-proxy-acces-refuse.png)

## Sécurité : portée et avis bienvenus

**Je ne peux pas affirmer que cette architecture est entièrement sécurisée.** Son évaluation dépend du périmètre, du modèle de menace et des tests réalisés ; d'autres personnes peuvent identifier des risques ou des failles que je n'ai pas observés. Suricata fonctionne ici en *détection* sur une copie du trafic et ne bloque pas les paquets ; aucun blocage « mini SOAR » n'est démontré. Je continue à revoir l'architecture à mesure que je progresse.

Si vous repérez une faille, une règle discutable ou une amélioration, vous pouvez [m'écrire](mailto:ihssanezaoui19@gmail.com?subject=Retour%20sur%20le%20projet%20Zero%20Trust) ou [ouvrir un ticket dans ce dépôt](https://github.com/ihssanezaoui19-star/Architecture-R-seau-S-curis-e-/issues) ; merci d'éviter d'y publier des secrets ou des données d'autrui.

## Documentation

[Lire la méthodologie pas à pas : décisions, configurations, tests, corrections et limites](METHODOLOGIE.md).

**Confidentialité :** le rapport technique original déjà présent dans ce dépôt porte la mention « Confidentiel — Usage interne uniquement » et contient des détails de laboratoire. Les nouveaux visuels de cette page sont recadrés ou reconstruits sans nom d'organisation, adresse, compte ou secret ; l'ancien fichier et l'historique nécessitent une revue séparée avant diffusion.
