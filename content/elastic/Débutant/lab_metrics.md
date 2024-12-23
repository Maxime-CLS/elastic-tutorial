---
title: "Lab - Metrics"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 10
---

Objectif : Dans ce laboratoire, vous explorerez la facilité avec laquelle vous pouvez obtenir des métriques système et NGINX. Vous explorerez également les tableaux de bord Kibana et verrez avec quelle facilité vous pouvez surveiller ces métriques.

## Infrastructure

Lancer Kibana pour commencer à explorer les métriques qui ont été inclus dans le processus de configuration.

Accédez à Observability > Infrastructure via le menu principal

![image.png](/elastic-tutorial/images/attachments/debutant/metrics/menu.png)


La vue Infrastructure dans Elastic Observability permet de surveiller et d'analyser les métriques de votre infrastructure en temps réel. Voici ce que vous y trouverez :

Inventaire de l'infrastructure : Une vue basée sur les métriques de l'ensemble de votre infrastructure, regroupée par les ressources surveillées, comme les serveurs, les conteneurs Docker, et les pods Kubernetes1.

Explorateur de métriques : Un outil pour créer des visualisations de séries temporelles basées sur l'agrégation de vos métriques, permettant de comparer différentes métriques et de les décomposer par champ2.

Vue des hôtes : Une interface conviviale pour visualiser les métriques des hôtes, aidant à diagnostiquer les pics problématiques et à identifier les utilisations élevées des ressources2.

Ces fonctionnalités vous aident à obtenir une vue complète et détaillée de la performance et de la santé de votre infrastructure.

![image.png](/elastic-tutorial/images/attachments/debutant/metrics/inventory.png)

### Host

En cliquant sur un hôte dans la vue Infrastructure d'Elastic Observability, vous pouvez observer les éléments suivants :

- Détails des métriques : Une vue détaillée des métriques de l'hôte, incluant l'utilisation du CPU, de la mémoire, des disques, et des réseaux.
- Historique des performances : Visualisation des tendances de performance sur différentes périodes pour identifier les anomalies et les pics d'utilisation.
- Informations contextuelles : Métadonnées et informations supplémentaires sur l'hôte, telles que le système d'exploitation, les versions des logiciels, et les configurations réseau.

Ces fonctionnalités permettent une analyse approfondie et une gestion efficace de la performance et de la santé de vos hôtes.

![image.png](/elastic-tutorial/images/attachments/debutant/metrics/inventory-hosts.png)

### Docker

Maintenant appliquez un filtre pour observer uniquement les conteneurs Docker :

![image.png](/elastic-tutorial/images/attachments/debutant/metrics/inventory-filter.png)

![image.png](/elastic-tutorial/images/attachments/debutant/metrics/inventory-docker.png)

En cliquant sur un conteneur dans la vue Infrastructure d'Elastic Observability, vous pouvez observer les éléments suivants :

 - Détails des métriques : Une vue détaillée des métriques du conteneur, incluant l'utilisation du CPU, de la mémoire, des E/S disque, et des réseaux.
 - Logs associés : Accès direct aux logs pertinents pour le conteneur sélectionné, facilitant le diagnostic des problèmes.
 - Historique des performances : Visualisation des tendances de performance sur différentes périodes pour identifier les anomalies et les pics d'utilisation.
 - Informations contextuelles : Métadonnées et informations supplémentaires sur le conteneur, telles que l'image Docker utilisée, les labels, et les configurations réseau.

Ces fonctionnalités permettent une analyse approfondie et une gestion efficace de la performance et de la santé de vos conteneurs Docker.

![image.png](/elastic-tutorial/images/attachments/debutant/metrics/metrics-nginx-overview.png)
![image.png](/elastic-tutorial/images/attachments/debutant/metrics/metrics-nginx-metadata.png)
![image.png](/elastic-tutorial/images/attachments/debutant/metrics/metrics-nginx-metrics.png)
![image.png](/elastic-tutorial/images/attachments/debutant/metrics/metrics-nginx-logs.png)


## Dashboard 

### Docker

Lancer Kibana pour commencer à explorer les métriques qui ont été inclus dans le processus de configuration.

Accédez à Analytics > Dashboard via le menu principal

![image.png](/elastic-tutorial/images/attachments/debutant/metrics/menu-dashboard.png)

Recherche un tableau de bord pour Docker : 

![image.png](/elastic-tutorial/images/attachments/debutant/metrics/dashboard-search.png)

Le tableau de bord [Metrics Docker] Overview dans l'intégration Docker d'Elastic fournit une vue d'ensemble des performances et de l'état de vos conteneurs Docker. Voici ce que vous y trouverez :

 - Utilisation du CPU : Visualisation de l'utilisation du CPU par conteneur, permettant d'identifier les conteneurs gourmands en ressources1.
 - Utilisation de la mémoire : Suivi de la consommation de mémoire pour chaque conteneur, aidant à détecter les fuites de mémoire et les usages excessifs2.
 - E/S disque : Surveillance des opérations d'entrée/sortie disque, utile pour comprendre les performances de stockage1.
 - Trafic réseau : Analyse du trafic réseau entrant et sortant des conteneurs, permettant de surveiller la bande passante utilisée2.
 - Statut des conteneurs : Informations sur l'état de santé et le statut des conteneurs, facilitant la gestion et le dépannage1.

Ce tableau de bord centralise les métriques essentielles pour une gestion efficace et une surveillance proactive de vos environnements Docker.

![image.png](/elastic-tutorial/images/attachments/debutant/metrics/dashboard-docker.png)


