---
title: "Lab - Synthetics"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 20
---

Objectif : Dans ce laboratoire, vous explorerez la facilité avec laquelle vous pouvez obtenir des métriques des disponibilités des services. Vous explorerez également les tableaux de bord Kibana et verrez avec quelle facilité vous pouvez surveiller ces métriques.

## Expérience Utilisateur 

Naviguer dans l'application Petclinic quelque instant pour générer un traffic utilisateur sur un navigateur web.

Accédez à Observability > User Experience via le menu principal

![image.png](/elastic-tutorial/images/attachments/debutant/synthetics/menu.png)

La fonctionnalité User Experience dans Elastic Observability permet de quantifier et d'analyser la performance perçue de vos applications web en conditions réelles. Voici ce que vous y trouverez :

 - Métriques de performance : Collecte des données sur des indicateurs clés comme le temps de chargement des pages, la stabilité visuelle et l'interactivité, en utilisant les Core Web Vitals de Google.
 - Analyse détaillée : Permet de filtrer et d'explorer les données par URL, système d'exploitation, navigateur et localisation pour comprendre comment différents utilisateurs vivent l'expérience de votre site.
 - Surveillance en temps réel : Utilise l'agent Real User Monitoring (RUM) pour capturer les métriques de performance à chaque visite de page, offrant une vue précise et actuelle de l'expérience utilisateur1.

Cette fonctionnalité aide à identifier et à résoudre les problèmes de performance, améliorant ainsi l'expérience globale des utilisateurs de votre application web.

![image.png](/elastic-tutorial/images/attachments/debutant/synthetics/user-experience.png)


## Synthetics 

### Sonde de surveillance 

Accédez à Observability > Synthetics via le menu principal

![image.png](/elastic-tutorial/images/attachments/debutant/synthetics/menu.png)

La vue Synthetics Overview dans Elastic Observability offre une vue d'ensemble des performances et de la disponibilité de vos services web. Voici ce que vous y trouverez :

 - Statut des moniteurs : Un aperçu du statut actuel de tous les moniteurs, indiquant si les services sont opérationnels ou rencontrent des problèmes.
 - Erreurs et alertes : Le nombre d'erreurs survenues au cours des dernières heures et les alertes générées, permettant de détecter rapidement les problèmes.
 - Détails des moniteurs : Informations sur chaque moniteur, y compris la localisation, le type de moniteur (HTTP, TCP, ICMP, navigateur), et la durée moyenne des tests.
 - Filtres et recherche : Options pour filtrer les moniteurs par statut, type, localisation, et plus encore, facilitant la navigation et l'analyse.

Cette vue permet de surveiller efficacement la disponibilité et les performances de vos services, en identifiant rapidement les problèmes potentiels.

Accédez à Management > Nginx Availability via les onglets 

![image.png](/elastic-tutorial/images/attachments/debutant/synthetics/synthetics-nginx.png)

La vue d'un moniteur HTTP dans Elastic Observability permet de surveiller la disponibilité et les performances de vos endpoints HTTP. Voici ce que vous y trouverez :

 - Statut des requêtes : Affiche le statut des requêtes HTTP (succès, échec) pour chaque point de terminaison surveillé.
 - Temps de réponse : Visualisation des temps de réponse des requêtes, permettant d'identifier les lenteurs et les pics de latence.
 - Détails des erreurs : Informations détaillées sur les erreurs rencontrées, y compris les codes de statut HTTP et les messages d'erreur.
 - Historique des vérifications : Un historique des vérifications effectuées, montrant les tendances et les variations de performance au fil du temps1.

Ces fonctionnalités aident à assurer la fiabilité et la performance de vos services web en identifiant rapidement les problèmes potentiels.

Retourner dans la vue Management et sélectionner "Home -> Show Vets journey"

![image.png](/elastic-tutorial/images/attachments/debutant/synthetics/synthetics-browser.png)

La vue d'un moniteur navigateur (browser monitor) dans Elastic Observability permet de surveiller les performances et la disponibilité de vos applications web en simulant les interactions des utilisateurs. Voici ce que vous y trouverez :

 - Statut des tests : Affiche le statut des tests de navigation (succès, échec) pour chaque parcours utilisateur simulé.
 - Temps de chargement des pages : Visualisation des temps de chargement des différentes pages visitées, aidant à identifier les lenteurs.
 - Détails des erreurs : Informations détaillées sur les erreurs rencontrées, y compris les messages d'erreur et les étapes du parcours où elles se produisent.
 - Captures d'écran et vidéos : Captures d'écran et enregistrements vidéo des sessions de test, permettant de visualiser exactement ce que les utilisateurs voient.

Ces fonctionnalités aident à garantir que vos applications web fonctionnent correctement et offrent une expérience utilisateur optimale.



## A vous de jouez !

On peut constater au travers des autres parcours utilisateurs que l'application n'a pas le comportement attendu. Proposer moi une analyse permettant de comprendre l'origine du problème grace a la solution Elastic. Et proposer moi une solution pour corriger le dysfonctionnement.


L'analyse doit suivre le format d'un post mortem. Voici un exemple de post mortem : https://sre.google/sre-book/example-postmortem/
