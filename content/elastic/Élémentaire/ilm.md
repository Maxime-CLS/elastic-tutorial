---
title: "Lab - Index Lifecycle Policy"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 10
---

Vous pouvez configurer des politiques de gestion du cycle de vie des index (ILM) pour gérer automatiquement les index en fonction de vos exigences en matière de performances, de résilience et de rétention. Par exemple, vous pouvez utiliser ILM pour :

* Faire tourner un nouvel index lorsqu'un index atteint une certaine taille ou un certain nombre de documents.
* créer un nouvel index chaque jour, semaine ou mois et archiver les précédents
* supprimer les index périmés pour appliquer les normes de conservation des données.

Vous pouvez créer et gérer les politiques de cycle de vie des index via Kibana Management ou les API ILM. Lorsque vous activez la gestion du cycle de vie des index pour Beats ou le plugin de sortie Logstash Elasticsearch, des politiques par défaut sont configurées automatiquement.

La mise en place de règle de gestion permet :

* Augmenter les **performances** de recherche et/ou de visualisation
* Obtenir une meilleure **résilience** de ses données
* De définir des standards de **rétention**.

L'avantage de la fonctionnalité ILM est d'automatiser cette gestion avec des règles simple d'usage.

![ILM](https://www.elastic.co/guide/en/elasticsearch/reference/current/images/ilm/index-lifecycle-policies.png)

### ILM Settings

Par défaut sur votre cluster la fonctionnalité ILM est configuré à un interval de vérification de 10 minutes.

Ce paramètre peut être modifié dans des contexts **hors prod**.

```
GET _cluster/settings

PUT _cluster/settings
{
	"persistent":{
		"indices.lifecycle.poll_interval": "1s"
	}
}
```

```
{
  "acknowledged" : true,
  "persistent" : {
    "indices" : {
      "lifecycle" : {
        "poll_interval" : "1s"
      }
    }
  },
  "transient" : { }
}
```

### ILM Policy

La définition d'une règle intègre plusieurs paramètres :

* Phase : Il existe plusieurs type de phase hot, warm et cold.
* Rollover : Le roulement d'un index s'applique avec au moins un de ces paramètres : max_age, max_size, max_docs.
* Delete : La rétention d'un index avec la fonction de suppression.

Il est recommander d'effectuer dans la phase **hot** une rotation d'index sur les paramètres suivant :
* Logs :
    * max_age : 15d
    * max_size : 45gb
* Metric :
    * max_age : 15d
    * max_size : 40gb

Et une deuxième recommandation est d'effectuer une retention d'index égale à 90 jours.

Cela permet d'augmenter les performances de l'index et d'optimiser les recherches.

Pour notre besoin d'aujourd'hui, nous allons préparer une ILM avec une phase très courte. Ouvrez dans le menu Kibana le DevTool.

```
PUT _ilm/policy/observability-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_docs": 10
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "delete": {
        "min_age": "60d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

```
{
  "acknowledged" : true
}
```

### Components Template
#### Settings

Nous allons configurer des paramètres par défaut pour tous les index ELasticsearch qui seront créé.

Il est recommandé d'avoir les paramètres de réplications suivant :
* shard : 1
* replicas : 1

```
PUT _component_template/observability-settings
{
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 10,
      "index.lifecycle.name": "observability-policy"
    }
  },
  "_meta": {
    "description": "Settings for Observability data"
  }
}
```

```
{
  "acknowledged" : true
}
```


#### Mapping

NOus allons configurer un mapping des champs par défaut pour toutes les données d'observabilité.

```
PUT _component_template/observability-mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date",
          "format": "date_optional_time||epoch_millis"
        },
        "message": {
          "type": "wildcard"
        }
      }
    }
  },
  "_meta": {
    "description": "Mappings for @timestamp and message fields"
  }
}
```

```
{
  "acknowledged" : true
}
```


### Index Template

L'index template est une fonctionnalité permettant de définir des paramètres par défaut à vos index.
Cela permet d'automatiser la gestion des index en appliquant des configurations de réplications, de règles de gestion automatiquement.


```
PUT _index_template/observability-template
{
  "index_patterns": ["observability-*"],
  "data_stream": { },
  "composed_of": [ "observability-mappings", "observability-settings" ],
  "priority": 500,
  "_meta": {
    "description": "Template for my time series data"
  }
}
```

```
{
  "acknowledged" : true
}
```

### Créer votre Data Stream

```
PUT _data_stream/observability-time-series
```

```
{
  "acknowledged" : true
}
```


### Ajouter des données dans le Data Stream

```
PUT observability-time-series/_bulk
{"create":{}}
{"@timestamp":"2099-05-06T16:21:15.000Z","message":"192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"}
{"create":{}}
{"@timestamp":"2099-05-06T16:25:42.000Z","message":"192.0.2.255 - - [06/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638"}
{"create":{}}
{"@timestamp":"2099-05-06T16:21:15.000Z","message":"192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"}
{"create":{}}
{"@timestamp":"2099-05-06T16:25:42.000Z","message":"192.0.2.255 - - [06/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638"}
{"create":{}}
{"@timestamp":"2099-05-06T16:21:15.000Z","message":"192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"}
{"create":{}}
{"@timestamp":"2099-05-06T16:25:42.000Z","message":"192.0.2.255 - - [06/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638"}
{"create":{}}
{"@timestamp":"2099-05-06T16:21:15.000Z","message":"192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"}
{"create":{}}
{"@timestamp":"2099-05-06T16:25:42.000Z","message":"192.0.2.255 - - [06/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638"}
{"create":{}}
{"@timestamp":"2099-05-06T16:21:15.000Z","message":"192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"}
{"create":{}}
{"@timestamp":"2099-05-06T16:25:42.000Z","message":"192.0.2.255 - - [06/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638"}
{"create":{}}
{"@timestamp":"2099-05-06T16:21:15.000Z","message":"192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"}
{"create":{}}
{"@timestamp":"2099-05-06T16:25:42.000Z","message":"192.0.2.255 - - [06/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638"}
{"create":{}}
{"@timestamp":"2099-05-06T16:21:15.000Z","message":"192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"}
{"create":{}}
{"@timestamp":"2099-05-06T16:25:42.000Z","message":"192.0.2.255 - - [06/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638"}
{"create":{}}
{"@timestamp":"2099-05-06T16:21:15.000Z","message":"192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"}
{"create":{}}
{"@timestamp":"2099-05-06T16:25:42.000Z","message":"192.0.2.255 - - [06/May/2099:16:25:42 +0000] \"GET /favicon.ico HTTP/1.0\" 200 3638"}
```

```
{
  "took" : 12,
  "errors" : false,
  "items" : [
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "HggBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "HwgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "IAgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "IQgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 3,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "IggBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 4,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "IwgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 5,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "JAgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 6,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "JQgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 7,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "JggBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 8,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "JwgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 9,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "KAgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 10,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "KQgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 11,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "KggBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 12,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "KwgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 13,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "LAgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 14,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "create" : {
        "_index" : ".ds-observability-time-series-2022.02.21-000001",
        "_type" : "_doc",
        "_id" : "LQgBHn8BzEvhAp6LaS34",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 15,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}
```



### Mettre à jours les données Data Stream

```
POST observability-time-series/_doc
{
  "@timestamp": "2099-05-06T16:21:15.000Z",
  "message": "192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"
}
```
```
{
  "_index" : ".ds-observability-time-series-2022.02.21-000002",
  "_type" : "_doc",
  "_id" : "ZQgCHn8BzEvhAp6LozUb",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

On constate que le nom de l'index est différent **.ds-observability-time-series-2022.02.21-000002**, il y a eu une rotation automatique de l'index. On constate alors que la politique de rotation est bien appliquée.

### ILM Check

```
GET _ilm/status
```

```
{
  "operation_mode" : "RUNNING"
}
```

```
GET _ilm/policy/observability-policy
```

```
{
  "observability-policy" : {
    "version" : 1,
    "modified_date" : "2022-02-21T20:35:02.304Z",
    "policy" : {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_docs" : 10
            },
            "set_priority" : {
              "priority" : 50
            }
          }
        },
        "delete" : {
          "min_age" : "60d",
          "actions" : {
            "delete" : {
              "delete_searchable_snapshot" : true
            }
          }
        }
      }
    },
    "in_use_by" : {
      "indices" : [
        ".ds-observability-time-series-2022.02.21-000001",
        ".ds-observability-time-series-2022.02.21-000002"
      ],
      "data_streams" : [
        "observability-time-series"
      ],
      "composable_templates" : [
        "observability-template"
      ]
    }
  }
}
```

On retrouve la configuration de notre ILM ainsi que les index rattachés.

```
GET observability-time-series/_ilm/explain
```

```
{
  "indices" : {
    ".ds-observability-time-series-2022.02.21-000001" : {
      "index" : ".ds-observability-time-series-2022.02.21-000001",
      "managed" : true,
      "policy" : "observability-policy",
      "lifecycle_date_millis" : 1645475884407,
      "age" : "4.91m",
      "phase" : "hot",
      "phase_time_millis" : 1645475859767,
      "action" : "complete",
      "action_time_millis" : 1645475885210,
      "step" : "complete",
      "step_time_millis" : 1645475885210,
      "phase_execution" : {
        "policy" : "observability-policy",
        "phase_definition" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_docs" : 10
            },
            "set_priority" : {
              "priority" : 50
            }
          }
        },
        "version" : 1,
        "modified_date_in_millis" : 1645475702304
      }
    },
    ".ds-observability-time-series-2022.02.21-000002" : {
      "index" : ".ds-observability-time-series-2022.02.21-000002",
      "managed" : true,
      "policy" : "observability-policy",
      "lifecycle_date_millis" : 1645475884525,
      "age" : "4.91m",
      "phase" : "hot",
      "phase_time_millis" : 1645475884809,
      "action" : "rollover",
      "action_time_millis" : 1645475885210,
      "step" : "check-rollover-ready",
      "step_time_millis" : 1645475885210,
      "phase_execution" : {
        "policy" : "observability-policy",
        "phase_definition" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_docs" : 10
            },
            "set_priority" : {
              "priority" : 50
            }
          }
        },
        "version" : 1,
        "modified_date_in_millis" : 1645475702304
      }
    }
  }
}
```

On peut observer les différentes informations de nos index concernant ILM appliquée deçu.


Résumé : Dans ce laboratoire, vous avez exploré la facilité avec laquelle vous pouvez mettre en place une politique de gestion du cylce de vie de vos index. Vous avez également exploré les différentes composants d'une politique de gestion d'index au travers des settings, mappings, template et data stream.
