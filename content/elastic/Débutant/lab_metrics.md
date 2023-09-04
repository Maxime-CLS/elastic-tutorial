---
title: "Lab - Metrics"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 10
---

Objectif : Dans ce laboratoire, vous explorerez la facilité avec laquelle vous pouvez obtenir des métriques système et NGINX avec Metricbeat et les indexer dans Elasticsearch. Vous explorerez également les tableaux de bord Kibana et verrez avec quelle facilité vous pouvez surveiller ces métriques.

Voici le fichier de configuration de l'agent de collecte pour les données de type métriques.

Cette configuration va nous permettre :
- Collecter les métriques de la machine Linux
- Collecter les métriques de containers Docker
- Contextualiser les données collecter au travers de metadata
- Envoyer les données à un cluster Elasticsearch
- Provisionner des tableaux de bords


**metricbeat.yml**

```
#-------------------------------- Autodiscovery -------------------------------
# Autodiscover allows you to detect changes in the system and spawn new modules as they happen.
metricbeat.autodiscover:
  providers:
    - type: docker
      # https://www.elastic.co/guide/en/beats/metricbeat/current/configuration-autodiscover-hints.html
      hints.enabled: true

metricbeat.modules:
#------------------------------- System Module -------------------------------
- module: system
  metricsets: ["cpu", "load", "memory", "network", "process", "process_summary", "core", "diskio", "socket"]
  processes: ['.*']
  process.include_top_n:
    by_cpu: 5
    by_memory: 5
  period: 10s
  cpu.metrics:  ["percentages"]
  core.metrics: ["percentages"]

- module: system
  period: 1m
  metricsets:
    - filesystem
    - fsstat
  processors:
  - drop_event.when.regexp:
      system.filesystem.mount_point: '^/(sys|cgroup|proc|dev|etc|host|lib)($|/)'

- module: system
  period: 15m
  metricsets:
    - uptime

#------------------------------- Docker Module -------------------------------
- module: docker
  metricsets: ["container", "cpu", "diskio", "healthcheck", "info", "memory", "network"]
  hosts: ["unix:///var/run/docker.sock"]
  period: 10s

#================================ Processors ===================================
processors:
- add_docker_metadata: ~
- add_locale:
    format: offset
- add_host_metadata:
    netinfo.enabled: true

#========================== Elasticsearch output ===============================
output.elasticsearch:
  hosts: ["https://es01:9200"]
  username: "elastic"
  password: "CHANGEME"
  ssl.enabled: true
  ssl.certificate_authorities: ["/usr/share/elasticsearch/config/certificates/ca/ca.crt"]

#============================== Dashboards =====================================
setup.dashboards:
  enabled: true

#============================== Kibana =========================================
setup.kibana:
  host: "https://kib01:5601"
  username: "elastic"
  password: "CHANGEME"
  ssl.enabled: true
  ssl.certificate_authorities: ["/usr/share/elasticsearch/config/certificates/ca/ca.crt"]
```


Maintenant, ouvrez une nouvelle fenêtre de terminal et exécutez le script suivant pour simuler la charge sur votre serveur NGINX.

```
cd elastic-platform/app/
./artificial_load.sh
```

Lancez Kibana pour commencer à explorer les tableaux de bord qui ont été inclus dans le processus de configuration.

Ensuite, accédez à Dashboard via le menu principal.

![image.png](/elastic-tutorial/images/attachments/debutant/dashboard-menu.png)

Recherchez le système et ouvrez la vue d'ensemble de Metricbeat System ECS.

![image.png](/elastic-tutorial/images/attachments/debutant/metricbeat-dashboards.png)

Vous verrez quelque chose de similaire à la page ci-dessous.

![image.png](/elastic-tutorial/images/attachments/debutant/metricbeat-system-dashboard.png)

Cliquez sur Vue d'ensemble de l'hôte en haut du tableau de bord actuel pour voir un tableau de bord avec plus d'informations que Metricbeat a collectées à partir de l'hôte de votre environnement de laboratoire.

![image.png](/elastic-tutorial/images/attachments/debutant/metricbeat-host-overview-dashboard.png)


Retournez sur la page des tableaux de bord Kibana, recherchez nginx et ouvrez le tableau de bord ECS [Metricbeat Nginx] Overview.

![image.png](/elastic-tutorial/images/attachments/debutant/nginx-dashboards.png)

Vous verrez quelque chose de similaire à la page ci-dessous.

![image.png](/elastic-tutorial/images/attachments/debutant/metricbeat-nginx-dashboard.png)

Retournez dans les tableaux de bord Kibana et cliquez sur Créer un nouveau tableau de bord pour comparer les données NGINX collectées à partir de Filebeat et Metricbeat.

![image.png](/elastic-tutorial/images/attachments/debutant/filebeat-metricbeat-1.png)

Cliquez sur Ajouter depuis la bibliothèque pour commencer à ajouter des visualisations à votre tableau de bord.

![image.png](/elastic-tutorial/images/attachments/debutant/filebeat-metricbeat-2.png)

Utilisez la barre de requête pour filtrer par visualisations filebeat nginx et ajoutez les visualisations Filebeat suivantes : Journaux d'accès dans le temps et Codes de réponse dans le temps.

![image.png](/elastic-tutorial/images/attachments/debutant/filebeat-metricbeat-3.png)

Maintenant, filtrez par visualisations metricbeat nginx pour ajouter les visualisations Metricbeat suivantes : Taux de requêtes et Taux de lecture/écriture/attente.

![image.png](/elastic-tutorial/images/attachments/debutant/filebeat-metricbeat-4.png)

Fermez la boîte de dialogue Ajouter à partir de la bibliothèque et enregistrez votre tableau de bord sous le nom de Filebeat x Metricbeat.

![image.png](/elastic-tutorial/images/attachments/debutant/filebeat-metricbeat-5.png)

Vérifiez les différentes informations que vous pouvez obtenir à partir des journaux et des métriques de NGINX dans le tableau de bord que vous venez de créer.

![image.png](/elastic-tutorial/images/attachments/debutant/filebeat-metricbeat-6.png)

Notez que vous pouvez utiliser les données de Filebeat et de Metricbeat pour vérifier le nombre de requêtes par seconde, comme le montrent les visualisations Access logs over time et Request Rate. La version open source de NGINX ne fournit pas de métriques sur les codes de réponse, mais vous pouvez utiliser Filebeat pour obtenir ces informations à partir des journaux NGINX, comme vous pouvez le voir dans la visualisation des codes de réponse dans le temps. Avec les journaux NGINX, vous pouvez également vérifier l'emplacement, le système d'exploitation, le navigateur, la durée et de nombreuses autres informations sur les demandes. Cependant, les journaux NGINX ne vous disent pas combien de connexions ont été acceptées, traitées et abandonnées. Dans ce cas, vous pouvez utiliser Metricbeat pour collecter ces informations ainsi que le nombre de connexions actives et le nombre de connexions en lecture, écriture et attente, comme vous pouvez le voir dans la visualisation du taux de lecture / écriture / attente.

Résumé : Dans ce laboratoire, vous avez exploré la facilité avec laquelle vous pouvez obtenir des mesures du système et de NGINX avec Metricbeat et les indexer dans Elasticsearch. Vous avez également exploré les tableaux de bord Kibana et vu avec quelle facilité vous pouvez surveiller ces mesures.
