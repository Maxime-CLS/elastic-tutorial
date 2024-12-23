---
title: "Lab - Logs"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 5
---

Objectif : Dans ce laboratoire, vous explorerez la facilité avec laquelle vous pouvez lire les fichiers journaux NGINX avec Kibana.

## Explore

Lancer Kibana pour commencer à explorer les logs qui ont été inclus dans le processus de configuration.

Accédez à Observability > Logs via le menu principal

![image.png](/elastic-tutorial/images/attachments/debutant/logs/menu.png)

La console Explore dans Observability Logs d'Elastic permet de rechercher, filtrer et analyser les logs en temps réel. Voici ce que vous y trouverez :

 - Recherche et filtrage : Accédez rapidement à vos logs en utilisant des filtres et des recherches pour trouver les informations nécessaires.
 - Visualisation des données : Affichez les résultats de vos recherches sous forme de visualisations pour une meilleure compréhension des données.
 - Personnalisation : Enregistrez et personnalisez vos recherches, et ajoutez-les à des tableaux de bord pour un accès rapide.

Cette console centralise la gestion des logs, facilitant ainsi le suivi et l'analyse des événements dans vos systèmes.

![image.png](/elastic-tutorial/images/attachments/debutant/logs/explore.png)

### Statistics

La vue Statistics dans l'onglet Explore d'Elastic Observability permet d'analyser les données de logs de manière agrégée. Voici ce que vous y trouverez :

![image.png](/elastic-tutorial/images/attachments/debutant/logs/logs-statistics.png)

 - Aperçu des métriques : Visualisez des statistiques clés comme le nombre total de logs, les niveaux de sévérité, et les fréquences d'occurrence.
 - Distribution des logs : Examinez la répartition des logs par différentes dimensions, telles que les sources, les types d'événements, et les périodes de temps.
 - Tendances et anomalies : Identifiez les tendances et détectez les anomalies dans les données de logs pour une analyse approfondie.

Cette vue facilite la compréhension globale des logs et aide à identifier rapidement les problèmes potentiels.

### Filters

Ajoutez un filtre dans les données afin d'observer uniquement les événements de conteneur NGINX 

![image.png](/elastic-tutorial/images/attachments/debutant/logs/search-filter.png)


En cliquant sur une ligne de logs NGINX dans le menu Explore de Elastic Observability, vous pouvez observer les éléments suivants :

Détails du log : Une vue détaillée du log sélectionné, incluant toutes les informations pertinentes comme les timestamps, les adresses IP, les requêtes HTTP, et les codes de réponse.

Contexte supplémentaire : Des informations contextuelles sur l'événement logué, telles que les métadonnées et les champs personnalisés.

Ces fonctionnalités vous aident à analyser et à comprendre en profondeur les événements enregistrés par vos instances NGINX.

![image.png](/elastic-tutorial/images/attachments/debutant/logs/logs-nginx.png)

