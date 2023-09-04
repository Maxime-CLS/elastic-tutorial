---
title: "Lab - Uptime"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 20
---

Objectif : Dans ce laboratoire, vous explorerez la facilité avec laquelle vous pouvez obtenir des métriques des disponibilités des services avec Heratbeat et les indexer dans Elasticsearch. Vous explorerez également les tableaux de bord Kibana et verrez avec quelle facilité vous pouvez surveiller ces métriques.


Voici le fichier de configuration de l'agent de collecte pour les données de type logs.

Cette configuration va nous permettre :
- Surveiller l'état des connexions tcp, http de l'application
- Contextualiser les données collecter au travers de metadata


**hearbeat.yml**

```
heartbeat.monitors:
- type: http
  id: nginx-service-status
  name: Nginx Status
  schedule: '@every 10s'
  urls:
    - http://nginx:8081
  timeout: 30s
  check.response:
    status: 200
    body:
      - "Welcome"
- type: http
  id: petclinic-service-status
  name: Petclinic Status
  schedule: '@every 10s'
  urls:
      - http://petclinic:8080/metrics/prometheus
  timeout: 30s
  check.response:
    status: 200
    body:
      - "jvm_memory_committed_bytes"
- type: tcp
  id: mysql-service-status
  name: MySQL Status
  schedule: '@every 10s'
  hosts: ["tcp://mysql:3306"]

processors:
- add_cloud_metadata: ~
- add_docker_metadata: ~

output.elasticsearch:
  hosts: 'https://es01:9200'
  username: "elastic"
  password: "CHANGEME"
  ssl.certificate_authorities: ["/usr/share/elasticsearch/config/certificates/ca/ca.crt"]
```

Maintenant, ouvrez une nouvelle fenêtre de terminal et exécutez le script suivant pour simuler la charge sur votre serveur NGINX.

```
cd elastic-platform/app/
./artificial_load.sh
```

Naviguer dans l'application Petclinic quelque instant pour générer un traffic utilisateur sur un navigateur web.

Lancez Kibana pour commencer à explorer les tableaux de bord qui ont été inclus dans le menu **Observability**

Cliquez sur le bouton **View app**

![image.png](/elastic-tutorial/images/attachments/debutant/observability_uptime.png)

Vous pouvez observer que l'agent beat réalise 3 vérification de disponibilité. Un pour chaque service de l'application.

Cliquez sur le monitor **Nginx Status**

![image.png](/elastic-tutorial/images/attachments/debutant/uptime_monitors.png)

Vous avez maintenant un vue détaillée du service nginx comme les temps de réponses, la disponibilité, etc...

![image.png](/elastic-tutorial/images/attachments/debutant/uptime_nginx_monitor.png)


## A vous de jouez !

On peut constater au travers des autres parcours utilisateurs que l'application n'a pas le comportement attendu. Proposer moi une analyse permettant de comprendre l'origine du problème grace a la solution Elastic.
Et proposer moi une solution pour corriger le dysfonctionnement.




Résumé : Dans ce laboratoire, vous avez exploré la facilité avec laquelle vous pouvez obtenir des mesures des vos services liée à l'application Petclinic avec Heartbeat et les indexer dans Elasticsearch. Vous avez également exploré les tableaux de bord Kibana et vu avec quelle facilité vous pouvez surveiller ces mesures.

