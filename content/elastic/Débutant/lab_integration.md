---
title: "Lab - Integration"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 2
---

Objectif : Dans ce laboratoire, vous explorerez la facilité avec laquelle vous pouvez faciliter la collecte, l'analyse et la visualisation des données provenant de diverses sources.

Elastic Agent, lorsqu'il est configuré avec Fleet Server, permet une gestion centralisée des agents Elastic. Voici un aperçu de son fonctionnement :

 - Fleet Server : C'est un composant de l'Elastic Stack qui agit comme un serveur central pour gérer plusieurs connexions d'Elastic Agents. Il est responsable de la mise à jour des politiques des agents, de la collecte des informations de statut et de la coordination des actions entre les agents.

 - Elastic Agent : C'est un agent unifié qui collecte des données et les envoie à l'Elastic Stack. Il peut être configuré pour différentes intégrations afin de surveiller divers services et applications.

 - Intégrations : Les intégrations permettent de collecter des données spécifiques à partir de différentes sources (comme des applications ou des services) et de les envoyer à Elastic pour l'analyse. Chaque intégration inclut des configurations prédéfinies, des tableaux de bord et des visualisations pour faciliter l'observabilité.


![image.png](/elastic-tutorial/images/attachments/debutant/integrations/integrations.png)



Lancer Kibana pour commencer à explorer les intégrations qui ont été inclus dans le processus de configuration.

Accédez à Integration via le menu principal


![image.png](/elastic-tutorial/images/attachments/debutant/integrations/menu-integration.png)

Recherchez docker et ouvrez l'intégration.

![image.png](/elastic-tutorial/images/attachments/debutant/integrations/docker-integration.png)

L'intégration Docker dans Elastic permet de collecter des métriques et des logs des conteneurs Docker. Voici ce que vous trouverez dans le menu de cette intégration :

1. Métriques des conteneurs : Collecte des informations et des statistiques sur les conteneurs en cours d'exécution, y compris l'utilisation du CPU, de la mémoire, des E/S disque, et des réseaux.

2. Logs des conteneurs : Collecte des logs générés par les conteneurs Docker pour une analyse approfondie et un suivi des événements.


Cette intégration facilite la surveillance et l'observabilité des environnements Docker en centralisant les données dans l'Elastic Stack.

Accédez dans le menu de l'intégration à "Integration Policies"

![image.png](/elastic-tutorial/images/attachments/debutant/integrations/docker-policies-integration.png)

Cette vue permet d'avoir une vision globale des politiques de collectes utilisant l'intégration Docker.

Accédez dans le menu de l'intégration à "Assets"

![image.png](/elastic-tutorial/images/attachments/debutant/integrations/docker-assets-integration.png)

Dans l'onglet Assets de l'intégration Docker dans Elastic, vous trouverez les éléments suivants :

Tableaux de bord (Dashboards) : Des visualisations prédéfinies pour surveiller les performances et les logs des conteneurs Docker, facilitant l'analyse des données collectées.

Modèles d'index (Index Templates) : Des configurations pour structurer les données indexées dans Elasticsearch, optimisant le stockage et la recherche.

Pipelines d'ingestion (Ingest Pipelines) : Des définitions de pipelines pour traiter et transformer les données avant leur indexation dans Elasticsearch.

Ces assets permettent de configurer rapidement et efficacement la surveillance des environnements Docker avec l'Elastic Stack.

Dans ce Labs, nous utilisons 4 intégrations :
 - **System** : Cette intégration permet de collecter des métriques et des logs des serveurs et des ordinateurs personnels. Elle surveille l'utilisation du CPU, de la mémoire, des disques, et des réseaux, ainsi que les événements système.
 - **Fleet Sevrer** : Cette intégration centralise la gestion des Elastic Agents. Fleet Server met à jour les politiques des agents, collecte les informations de statut et coordonne les actions entre les agents.
 - **Docker** : Cette intégration collecte des métriques et des logs des conteneurs Docker. Elle surveille l'utilisation des ressources des conteneurs, comme le CPU, la mémoire, les E/S disque, et les réseaux.
 - **Elastic Synthetics** : : Cette intégration permet de créer des moniteurs synthétiques pour tester la disponibilité et les performances des services web. Elle utilise des moniteurs légers et des navigateurs réels pour simuler les interactions des utilisateurs.