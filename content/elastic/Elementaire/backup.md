---
title: "Lab - Backup & Restore"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 15
---


La haute disponibilité et la redondance d'Elasticsearch en font une plateforme stable et fiable pour le stockage de quantités massives de données. Toutefois, pour vous protéger contre les erreurs humaines et les catastrophes naturelles, vous devez toujours sauvegarder vos données Elasticsearch. L'endroit où vous stockez vos sauvegardes de données Elasticsearch est entièrement à votre discrétion. Dans ce laboratoire pratique, nous utiliserons le système de fichiers local pour démontrer comment :

* Créer des dépôts instantanés
* Sauvegarder des index spécifiques
* Restaurer des données à partir d'un instantané


#### Créez le référentiel "observability_repo".

Utilisez l'outil DevTool Kibana pour exécuter ce qui suit :

```
PUT _snapshot/observability_repo?verify=false
{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/snapshots/"
  }
}
```

#### Sauvegarde de l'index "observability-time-series"

Utilisez l'outil de console Kibana pour exécuter ce qui suit :

```
PUT _snapshot/observability_repo/observability?wait_for_completion=true
{
  "indices": "observability-time-series",
  "include_global_state": false
}
```

#### Restaurez l'index "observability-time-series" en tant que "observability-time-series_restored"

Utilisez l'outil de console Kibana pour exécuter ce qui suit :

```
POST _snapshot/observability_repo/observability/_restore
{
  "indices": "observability-time-series",
  "rename_pattern": "(.+)",
  "rename_replacement": "$1_restored"
}
```

```
{
  "accepted" : true
}
```

```
GET _cat/indices
```

```
green  open metricbeat-7.17.0-2022.02.22-000001                      WBLcTqYORtS5LJyFttVBXQ 1  1  22122    0  67.6mb  34.8mb
green  open apm-7.17.0-error-000001                                  TuCoez2IQP6xmRJl70TKvw 1  1   1842    0   4.7mb   2.3mb
yellow open .ds-observability-time-series-2022.02.22-000002          xKbIQkulR0mMwv8aCbR5Og 1 10      1    0  12.3kb   4.1kb
yellow open .ds-observability-time-series-2022.02.22-000001          i_sIi8P2SUu6mNmlsMjv3Q 1 10     16    0  19.4kb   6.5kb
green  open apm-7.17.0-transaction-000001                            O14qvTcvQyOOIp8kIGGpKg 1  1  17027    0    25mb  11.1mb
green  open .apm-agent-configuration                                 jkaqvfq0Sy25CHaLDswUUA 1  1      0    0    452b    226b
green  open apm-7.17.0-metric-000001                                 mXYNU6V9SU2tlAmODIjPHQ 1  1   4497    0   4.7mb   2.3mb
green  open apm-7.17.0-span-000001                                   iOVtWj8iRL6b-YbxROQ9ig 1  1  75766    0  44.5mb  19.8mb
green  open apm-7.17.0-onboarding-2022.02.22                         dJ8FT0LvSn2HTUynoxWtbQ 1  1      1    0  16.1kb     8kb
green  open apm-7.17.0-profile-000001                                dQEJ37KkSzmQr4DVKLQnAw 1  1      0    0    452b    226b
green  open filebeat-7.17.0-2022.02.22-000001                        hN3UWBgXRYm9KA66iHfayQ 1  1 236916    0 279.3mb 138.5mb
green  open .ds-observability-time-series-2022.02.22-000001_restored _WQSy8o3QTi6l4inCqpA8Q 1  1     16    0  13.1kb   6.5kb
green  open .security-7                                              o2VUY0seRxSefLHyOge4rQ 1  1     60    0 531.1kb 295.6kb
green  open .kibana_7.17.0_001                                       J_lf9M3zRjmHgMdRhwcqLw 1  1   4424  328   8.1mb     4mb
green  open .apm-custom-link                                         fnAtCfqXSsOCN7dqyGQT8A 1  1      0    0    452b    226b
green  open .ds-observability-time-series-2022.02.22-000002_restored SUVTggxqRMODlSz4pE3gEw 1  1      1    0   8.3kb   4.1kb
green  open heartbeat-7.17.0-2022.02.22-000001                       U50p8dOMSz2jhWz7kHRDnw 1  1    648    0   2.6mb   1.3mb
green  open .async-search                                            JjeGtgVgTYKlJ8PsvmwL7Q 1  1      0    0   6.9kb   3.4kb
green  open .kibana_task_manager_7.17.0_001                          x63yGuhDSaudkMeqtmxANA 1  1     18 1122 753.7kb 368.6kb
```

### SLM Policy

La gestion du cycle de vie des instantanés (SLM) est le moyen le plus simple de sauvegarder régulièrement un cluster. Une politique SLM prend automatiquement des instantanés selon une planification prédéfinie. La politique peut également supprimer les instantanés en fonction des règles de rétention que vous définissez.

```
PUT _slm/policy/daily-observability-snapshots
{
  "name": "<observability-{now/d}>",
  "schedule": "0 30 1 * * ?",
  "repository": "observability_repo",
  "config": {
    "indices": [
      "observability-time-series"
    ]
  },
  "retention": {
    "expire_after": "30d",
    "min_count": 15,
    "max_count": 30
  }
}
```

```
GET _slm/policy/daily-observability-snapshots
```

```
{
  "daily-observability-snpashots" : {
    "version" : 1,
    "modified_date_millis" : 1645547161680,
    "policy" : {
      "name" : "<observability-{now/d}>",
      "schedule" : "0 30 1 * * ?",
      "repository" : "observability_repo",
      "config" : {
        "indices" : [
          "observability-time-series"
        ]
      },
      "retention" : {
        "expire_after" : "30d",
        "min_count" : 15,
        "max_count" : 30
      }
    },
    "next_execution_millis" : 1645579800000,
    "stats" : {
      "policy" : "daily-observability-snpashots",
      "snapshots_taken" : 0,
      "snapshots_failed" : 0,
      "snapshots_deleted" : 0,
      "snapshot_deletion_failures" : 0
    }
  }
}
```

