---
title: "Lab - Alertes"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 5
---

Vous êtes le nouvel administrateur d'un cluster Elasticsearch à 3 nœuds. En prenant possession du cluster, vous constatez que aucune alerte n'est remonter sur l'état de la plateforme Elastic. Vos supérieurs vous demandent de donner la priorité à construire une solution d'alerting permettant de fournir des indicateurs de fiabilisés la plateforme.

Dans ce laboratoire pratique, vous aurez l'occasion de :

- Utiliser les API _security pour créer une API Key.
- utiliser l'API actions/connectors pour créer un connecteur de donnée
- Utiliser l'API alerting/rule pour mettre rapidement en place des alertes.


## Créer des alertes avec le module Rules & Connector

### Créer une API Key

Utilisez l'outil de console Kibana pour exécuter ce qui suit :

```
POST /_security/api_key
{
  "name": "my-api-key",
  "expiration": "30d",
  "metadata": {
    "application": "my-application",
    "environment": {
       "level": 1,
       "trusted": true,
       "tags": ["prod", "pprod"]
    }
  }
}
```

```
{
  "id" : "uiVdOn8BIwAgRd-L1SZa",
  "name" : "my-api-key",
  "expiration" : 1648543702335,
  "api_key" : "AuXI4K3aSZ2jDYUbycs-8g",
  "encoded" : "dWlWZE9uOEJJd0FnUmQtTDFTWmE6QXVYSTRLM2FTWjJqRFlVYnljcy04Zw=="
}
```

### Créer un connecteur Server Log

Utilisez l'outil POSTMAN pour exécuter ce qui suit :

```
curl --location --request POST 'https://localhost:5601/api/actions/connector' \
--header 'kbn-xsrf: true' \
--header 'Authorization: APIKey YnlSTU9uOEJJd0FnUmQtTE9PQXc6em52akIyQlNRdHlPc3R4ODhTNFg3QQ==' \
--header 'Content-Type: text/plain' \
--data-raw '{
  "name": "Monitoring: Write to Kibana log TEST",
  "config": {},
  "connector_type_id": ".server-log"
}'
```

### Récupérer l'ID du connecteur

```
curl --location --request GET 'https://localhost:5601/api/actions/connectors' \
--header 'kbn-xsrf: true' \
--header 'Authorization: APIKey YnlSTU9uOEJJd0FnUmQtTE9PQXc6em52akIyQlNRdHlPc3R4ODhTNFg3QQ==' \
--data-raw ''
```

```
{
        "id": "6c65f7f0-97a6-11ec-b236-41c39a1728b8",
        "name": "Monitoring: Write to Kibana log",
        "config": {},
        "connector_type_id": ".server-log",
        "is_preconfigured": false,
        "referenced_by_count": 1,
        "is_missing_secrets": false
}
```

Vous pouvez maintenant voir dans l'interface Kibana le connecteur :

![image.png](/elastic-tutorial/images/attachments/intermediare/connectors.png)


### Créer une alerte sur la consommation des ressources

#### CPU Usage

Dans l'outil POSTMAN changer la méthode d'authentification en Basic Auth et renseigner le login et password de l'utilisateur elastic.

```
curl --location --request POST 'https://localhost:5601/api/alerting/rule' \
--header 'kbn-xsrf: true' \
--header 'Authorization: Basic ZWxhc3RpYzpXc1dnVkFUaUUxSWxsd2tEUXlQbw==' \
--header 'Content-Type: text/plain' \
--data-raw '{
            "enabled": true,
            "tags": ["monitoring", "cpu"],
            "consumer": "monitoring",
            "name": "CPU Usage",
            "throttle": "1d",
            "schedule": {
                "interval": "1m"
            },
            "params": {
                "threshold": 85,
                "duration": "5m"
            },
            "rule_type_id": "monitoring_alert_cpu_usage",
            "notify_when": "onThrottleInterval",
            "actions": [
                {
                    "group": "default",
                    "id": "6c65f7f0-97a6-11ec-b236-41c39a1728b8",
                    "params": {
                        "message": "{{context.internalShortMessage}}"
                    }
                }
            ]
        }
'
```

#### Disk Usage

```
curl --location --request POST 'https://localhost:5601/api/alerting/rule' \
--header 'kbn-xsrf: true' \
--header 'Authorization: Basic ZWxhc3RpYzpXc1dnVkFUaUUxSWxsd2tEUXlQbw==' \
--header 'Content-Type: text/plain' \
--data-raw '{
  "enabled": true,
  "tags": [],
  "consumer": "monitoring",
  "name": "Disk Usage",
  "throttle": "1d",
  "schedule": {
    "interval": "1m"
  },
  "params": {
    "threshold": 80,
    "duration": "5m"
  },
  "rule_type_id": "monitoring_alert_disk_usage",
  "notify_when": "onThrottleInterval",
  "actions": [
    {
      "group": "default",
      "id": "6c65f7f0-97a6-11ec-b236-41c39a1728b8",
      "params": {
        "message": "{{context.internalShortMessage}}"
      }
    }
  ]
}'
```


#### Experiation de la licence

```
curl --location --request POST 'https://localhost:5601/api/alerting/rule' \
--header 'kbn-xsrf: true' \
--header 'Authorization: Basic ZWxhc3RpYzpXc1dnVkFUaUUxSWxsd2tEUXlQbw==' \
--header 'Content-Type: text/plain' \
--data-raw '{
  "enabled": true,
  "tags": [],
  "consumer": "monitoring",
  "name": "License expiration",
  "throttle": "1d",
  "schedule": {
    "interval": "1m"
  },
  "params": {},
  "rule_type_id": "monitoring_alert_license_expiration",
  "notify_when": "onThrottleInterval",
  "actions": [
    {
      "group": "default",
      "id": "6c65f7f0-97a6-11ec-b236-41c39a1728b8",
      "params": {
        "message": "{{context.internalShortMessage}}"
      }
    }
  ]
}'
```

#### Etat de santé du cluster

```

curl --location --request POST 'https://localhost:5601/api/alerting/rule' \
--header 'kbn-xsrf: true' \
--header 'Authorization: Basic ZWxhc3RpYzpXc1dnVkFUaUUxSWxsd2tEUXlQbw==' \
--header 'Content-Type: text/plain' \
--data-raw '{
  "enabled": true,
  "tags": [],
  "consumer": "monitoring",
  "name": "Cluster health",
  "throttle": "1d",
  "schedule": {
    "interval": "1m"
  },
  "params": {},
  "rule_type_id": "monitoring_alert_cluster_health",
  "notify_when": "onThrottleInterval",
  "actions": [
    {
      "group": "default",
      "id": "6c65f7f0-97a6-11ec-b236-41c39a1728b8",
      "params": {
        "message": "{{context.internalShortMessage}}"
      }
    }
  ]
}'
```

#### Thread pool write rejections

```
curl --location --request POST 'https://localhost:5601/api/alerting/rule' \
--header 'kbn-xsrf: true' \
--header 'Authorization: Basic ZWxhc3RpYzpXc1dnVkFUaUUxSWxsd2tEUXlQbw==' \
--header 'Content-Type: text/plain' \
--data-raw '{
  "enabled": true,
  "tags": [],
  "consumer": "monitoring",
  "name": "Thread pool write rejections",
  "throttle": "1d",
  "schedule": {
    "interval": "1m"
  },
  "params": {
    "threshold": 300,
    "duration": "5m"
  },
  "rule_type_id": "monitoring_alert_thread_pool_write_rejections",
  "notify_when": "onThrottleInterval",
  "actions": [
    {
      "group": "default",
      "id": "6c65f7f0-97a6-11ec-b236-41c39a1728b8",
      "params": {
        "message": "{{context.internalShortMessage}}"
      }
    }
  ]
}'
```

#### Thread pool search rejections


```
curl --location --request POST 'https://localhost:5601/api/alerting/rule' \
--header 'kbn-xsrf: true' \
--header 'Authorization: Basic ZWxhc3RpYzpXc1dnVkFUaUUxSWxsd2tEUXlQbw==' \
--header 'Content-Type: text/plain' \
--data-raw '{
  "enabled": true,
  "tags": [],
  "consumer": "monitoring",
  "name": "Thread pool search rejections",
  "throttle": "1d",
  "schedule": {
    "interval": "1m"
  },
  "params": {
    "threshold": 300,
    "duration": "5m"
  },
  "rule_type_id": "monitoring_alert_thread_pool_search_rejections",
  "notify_when": "onThrottleInterval",
  "actions": [
    {
      "group": "default",
      "id": "6c65f7f0-97a6-11ec-b236-41c39a1728b8",
      "params": {
        "message": "{{context.internalShortMessage}}"
      }
    }
  ]
}'
```

#### Shard size

```
curl --location --request POST 'https://localhost:5601/api/alerting/rule' \
--header 'kbn-xsrf: true' \
--header 'Authorization: Basic ZWxhc3RpYzpXc1dnVkFUaUUxSWxsd2tEUXlQbw==' \
--header 'Content-Type: text/plain' \
--data-raw '{
  "enabled": true,
  "tags": [],
  "consumer": "monitoring",
  "name": "Shard size",
  "throttle": "12h",
  "schedule": {
    "interval": "1m"
  },
  "params": {
    "indexPattern": "*",
    "threshold": 55
  },
  "rule_type_id": "monitoring_shard_size",
  "notify_when": "onThrottleInterval",
  "actions": [
    {
      "group": "default",
      "id": "6c65f7f0-97a6-11ec-b236-41c39a1728b8",
      "params": {
        "message": "{{context.internalShortMessage}}"
      }
    }
  ]
}'
```


Vous pouvez maintenant voir dans l'interface Kibana les alertes :

![image.png](/elastic-tutorial/images/attachments/intermediare/rules.png)


Résumé : Dans ce laboratoire, vous avez exploré la facilité avec laquelle vous pouvez créer des connecteurs et des alertes et les indexer dans Elasticsearch.

