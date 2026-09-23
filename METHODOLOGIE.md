# Méthodologie — architecture réseau segmentée, démarche Zero Trust

> Projet personnel de laboratoire sur VMware Workstation. Le rapport technique et l'ancien README servent de sources. Les noms, adresses, comptes, certificats et secrets ne sont pas reproduits ; une configuration visible dans le rapport ne vaut pas preuve d'une sécurité globale.

## 1. Traduire le scénario en besoins d'accès

**Réalisé :** distinguer utilisateurs, invités, administration, serveurs, services en DMZ et accès VPN. Pour chaque zone, définir les services qui doivent être joignables et ceux qui doivent rester interdits, notamment la console OPNsense et les interfaces d'administration.

**Pourquoi :** la séparation des réseaux ne protège pas si une règle autorise finalement tout le trafic. La démarche Zero Trust retenue consiste à vérifier explicitement les accès nécessaires et à limiter les autres, puis à tester le résultat depuis le point de vue de chaque zone.

**Contrôle attendu :** établir une matrice de flux autorisés/refusés avant de modifier le pare-feu, pour éviter de casser DNS, proxy ou administration en appliquant un blocage trop large.

## 2. Mettre en place les réseaux virtuels et OPNsense

**Réalisé :** organiser les réseaux de test sous VMware et relier leurs interfaces à OPNsense. Configurer les interfaces, le routage inter-zones, le NAT et les règles de pare-feu nécessaires au scénario. Le proxy se trouve dans une zone séparée des postes et de l'administration.

**Pourquoi :** un point de filtrage commun permet de rendre visibles les échanges entre zones. La DMZ limite les accès du proxy à d'autres réseaux si ce service est compromis.

![Architecture simplifiée sans adresses](images/architecture-anonymisee.svg)

**Vérification documentée :** l'ancien README indique que la WebGUI OPNsense était initialement accessible depuis le réseau des utilisateurs (T-03), puis qu'une règle a été ajoutée pour réserver son accès à l'administration. Une revue complète exige de retester depuis **les deux zones** et de préserver un chemin d'administration pour corriger une mauvaise règle ; cette campagne complète n'est pas publiée.

## 3. Contrôler la sortie Web avec Squid

**Réalisé :** configurer Squid en DMZ, ses ACL, l'authentification et les journaux ; le rapport présente également des paramètres d'inspection TLS. Préparer dans OPNsense les règles qui dirigent la sortie Web souhaitée vers le proxy et empêchent un accès direct là où le scénario l'exige.

**Pourquoi :** une règle d'ACL sans journal ne permettrait pas d'expliquer un refus ; une sortie directe non prévue contournerait le proxy. L'inspection TLS et l'authentification peuvent, en revanche, casser des applications ou introduire des contraintes de confiance des certificats : vérifier l'effet avant de généraliser.

**Preuve disponible :** le rapport comporte une navigation autorisée et un refus d'accès ciblé par Squid ; la capture ci-dessous ne montre que ce refus, sans prouver que 100 % du trafic du laboratoire était forcé vers le proxy.

![Extrait recadré : accès refusé par Squid](images/capture-proxy-acces-refuse.png)

## 4. Limiter l'accès distant OpenVPN

**Réalisé :** préparer la chaîne de certificats, l'accès OpenVPN et l'authentification MFA/TOTP dans OPNsense. Affecter aux clients VPN des règles explicites correspondant aux ressources auxquelles ils doivent accéder.

**Pourquoi :** authentifier correctement un utilisateur ne suffit pas si le tunnel lui donne ensuite accès à tous les VLAN. L'ancien README consigne une règle « allow all » sur le VPN (T-09), supprimée au profit de règles ciblées.

**Impact et contrôle :** vérifier depuis un client VPN que les ressources légitimes restent joignables, tandis que la zone d'administration et la WebGUI ne le sont pas. Un blocage excessif interromprait le travail distant ; la documentation ne fournit pas une campagne exhaustive de ces essais après correction.

## 5. Centraliser les événements dans Wazuh

**Réalisé :** connecter les événements du pare-feu, du proxy, de Suricata et des postes à Wazuh ; utiliser Sysmon sur Windows pour enrichir les événements locaux. L'ancien README décrit un défaut du décodage Squid (T-07) : les journaux arrivaient, mais le nom d'utilisateur n'était pas extrait. Le décodeur a été modifié.

**Pourquoi :** pour relier un refus Web ou une alerte réseau à une session, l'adresse IP seule peut être insuffisante, notamment avec VPN ou adressage partagé. Sans journaux centralisés, reconstruire le parcours entre plusieurs zones prend plus de temps.

**Contrôle nécessaire :** générer une requête de test depuis un compte connu, retrouver dans Wazuh l'heure, l'action, l'adresse pertinente et l'identité attendue. Une règle de décodage configurée ne prouve pas que chaque événement sera correctement attribué.

## 6. Observer le trafic avec Suricata, puis corriger les règles

**Réalisé :** utiliser Suricata sur une copie du trafic, transférer ses alertes vers Wazuh et effectuer des essais contrôlés. L'ancien README consigne une détection partielle de scans et de scénarios de laboratoire (T-11 et T-12) avec les premières signatures ; les jeux de règles ont été mis à jour.

**Pourquoi :** une segmentation peut limiter des accès, mais n'explique pas toujours un trafic suspect sur les flux permis. Suricata apporte des alertes à examiner. Une mise à jour de signatures n'est pas une preuve de couverture : il faut rejouer chaque test, contrôler la présence de l'alerte et examiner les faux positifs.

**Limite :** en mode miroir, le capteur **ne bloque pas les paquets**. Le rapport contient un intitulé « mini SOAR », sans preuve d'un blocage automatique exécuté ; cette capacité n'est pas revendiquée.

## 7. Suivre les écarts, leurs corrections et l'impact

| Écart trouvé pendant les essais | Correction consignée | Pourquoi vérifier aussi l'effet sur le système |
| --- | --- | --- |
| T-03 : console OPNsense accessible aux utilisateurs | Règle de restriction à l'administration | Une règle mal ordonnée peut bloquer l'administrateur ou laisser un autre chemin ouvert |
| T-07 : identité absente du journal Squid dans Wazuh | Ajustement du décodeur | Une alerte mal attribuée peut égarer l'investigation |
| T-09 : accès VPN trop large | Suppression de « allow all » et règles ciblées | Il faut conserver les applications distantes utiles tout en fermant l'administration |
| T-11/T-12 : détections Suricata insuffisantes | Actualisation des signatures | Les détections et faux positifs doivent être vérifiés par de nouveaux essais |

Le rapport montre des réglages et des tests ponctuels ; l'ancien README rapporte ces corrections. Il manque une matrice complète avant/après avec preuves répétables pour chaque zone, mesure des faux positifs, effet sur les applications et procédures de retour arrière.

## 8. Évaluer honnêtement la démarche Zero Trust

Le modèle de menace, les flux retenus et les scénarios de test déterminent ce que la maquette permet de vérifier. **Je ne peux pas conclure qu'elle est entièrement sécurisée** : un autre regard peut révéler une règle trop large, une faiblesse d'authentification ou une faille que je n'ai pas observée. Les preuves citées concernent des configurations et essais précis, pas une garantie générale.

Les retours techniques sont utiles : [me contacter](mailto:ihssanezaoui19@gmail.com?subject=Retour%20sur%20le%20projet%20Zero%20Trust) ou [signaler une observation via GitHub Issues](https://github.com/ihssanezaoui19-star/Architecture-R-seau-S-curis-e-/issues), sans publier d'identifiants ou de données confidentielles.
