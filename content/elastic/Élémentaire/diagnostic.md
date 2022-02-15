---
title: "Lab - Diagnostic"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 20
---


Vous êtes le nouvel administrateur d'un cluster Elasticsearch à 3 nœuds. En prenant possession du cluster, vous constatez que plusieurs index sont dans un état jaune. Vos supérieurs vous demandent de donner la priorité au dépannage des problèmes d'allocation d'index avant toute autre tâche.

Lorsqu'ils sont configurés correctement, les clusters Elasticsearch sont hautement disponibles et tolérants aux pannes. Cela ne signifie pas nécessairement qu'ils sont imperméables aux pannes. L'erreur humaine et la défaillance matérielle sont toujours possibles. Le dépannage des problèmes de disponibilité des données sur un système distribué peut être un défi. Prenons le temps de démystifier certaines routines de dépannage de base lorsque vous remarquez des index jaunes dans votre cluster Elasticsearch. Dans ce laboratoire pratique, vous aurez l'occasion de :

- Utiliser les API _cat pour obtenir rapidement des informations vitales sur votre cluster.
- utiliser l'API _cluster/allocation/explain pour découvrir pourquoi un index est ou n'est pas alloué
- Utiliser l'API _settings pour mettre rapidement à jour les paramètres d'index.


### Résolution du problème de l'index "observability-time-series".

Utilisez l'outil de console Kibana pour exécuter ce qui suit :

```
GET _cat/indices?v
```

```
health status index                                                    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   metricbeat-7.17.0-2022.02.22-000001                      WBLcTqYORtS5LJyFttVBXQ   1   1     956609            0      1.8gb        915.3mb
green  open   apm-7.17.0-error-000001                                  TuCoez2IQP6xmRJl70TKvw   1   1       4994            0     19.9mb          9.9mb
yellow open   .ds-observability-time-series-2022.02.22-000002          xKbIQkulR0mMwv8aCbR5Og   1  10          1            0     12.5kb          4.1kb
yellow open   .ds-observability-time-series-2022.02.22-000001          i_sIi8P2SUu6mNmlsMjv3Q   1  10         16            0     19.6kb          6.5kb
green  open   apm-7.17.0-transaction-000001                            O14qvTcvQyOOIp8kIGGpKg   1   1     163280            0    130.7mb         62.3mb
green  open   apm-7.17.0-metric-000001                                 mXYNU6V9SU2tlAmODIjPHQ   1   1     265903            0       73mb         36.4mb
green  open   .apm-agent-configuration                                 jkaqvfq0Sy25CHaLDswUUA   1   1          0            0       452b           226b
green  open   apm-7.17.0-span-000001                                   iOVtWj8iRL6b-YbxROQ9ig   1   1     505334            0    377.8mb        153.4mb
green  open   apm-7.17.0-onboarding-2022.02.22                         dJ8FT0LvSn2HTUynoxWtbQ   1   1          1            0     16.1kb            8kb
green  open   apm-7.17.0-profile-000001                                dQEJ37KkSzmQr4DVKLQnAw   1   1          0            0       452b           226b
green  open   .security-7                                              o2VUY0seRxSefLHyOge4rQ   1   1         60            0    531.1kb        295.6kb
green  open   .ds-observability-time-series-2022.02.22-000001_restored _WQSy8o3QTi6l4inCqpA8Q   1   1         16            0     13.1kb          6.5kb
green  open   filebeat-7.17.0-2022.02.22-000001                        hN3UWBgXRYm9KA66iHfayQ   1   1    2453406            0      3.2gb          1.6gb
green  open   .kibana_7.17.0_001                                       J_lf9M3zRjmHgMdRhwcqLw   1   1       4439         1913      8.3mb          4.1mb
green  open   .apm-custom-link                                         fnAtCfqXSsOCN7dqyGQT8A   1   1          0            0       452b           226b
green  open   .ds-observability-time-series-2022.02.22-000002_restored SUVTggxqRMODlSz4pE3gEw   1   1          1            0      8.3kb          4.1kb
green  open   heartbeat-7.17.0-2022.02.22-000001                       U50p8dOMSz2jhWz7kHRDnw   1   1      25386            0     29.5mb         14.7mb
green  open   .async-search                                            JjeGtgVgTYKlJ8PsvmwL7Q   1   1          0            0      3.7kb           249b
green  open   .kibana_task_manager_7.17.0_001                          x63yGuhDSaudkMeqtmxANA   1   1         18        43080     15.2mb          7.6mb
```

```
GET _cat/nodes?v
```

```
ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
172.20.0.4           61          97  33    3.53    2.21     1.66 cdfhilmrstw *      es01
172.20.0.3           73          97  33    3.53    2.21     1.66 cdfhilmrstw -      es03
172.20.0.2           68          97  33    3.53    2.21     1.66 cdfhilmrstw -      es02
```

```
GET _cluster/allocation/explain
```

```
{
  "note" : "No shard was specified in the explain API request, so this response explains a randomly chosen unassigned shard. There may be other unassigned shards in this cluster which cannot be assigned for different reasons. It may not be possible to assign this shard until one of the other shards is assigned correctly. To explain the allocation of other shards (whether assigned or unassigned) you must specify the target shard in the request to this API.",
  "index" : ".ds-observability-time-series-2022.02.22-000001",
  "shard" : 0,
  "primary" : false,
  "current_state" : "unassigned",
  "unassigned_info" : {
    "reason" : "INDEX_CREATED",
    "at" : "2022-02-22T16:18:00.719Z",
    "last_allocation_status" : "no_attempt"
  },
  "can_allocate" : "no",
  "allocate_explanation" : "cannot allocate because allocation is not permitted to any of the nodes",
  "node_allocation_decisions" : [
    {
      "node_id" : "8T5zcuSbRbugyhTN5hxzHQ",
      "node_name" : "es02",
      "transport_address" : "172.20.0.2:9300",
      "node_attributes" : {
        "ml.machine_memory" : "8312127488",
        "ml.max_open_jobs" : "512",
        "xpack.installed" : "true",
        "ml.max_jvm_size" : "268435456",
        "transform.node" : "true"
      },
      "node_decision" : "no",
      "weight_ranking" : 1,
      "deciders" : [
        {
          "decider" : "same_shard",
          "decision" : "NO",
          "explanation" : "a copy of this shard is already allocated to this node [[.ds-observability-time-series-2022.02.22-000001][0], node[8T5zcuSbRbugyhTN5hxzHQ], [R], s[STARTED], a[id=rj9XVRi9Te2vqQgIIwsb8Q]]"
        }
      ]
    },
    {
      "node_id" : "QjZ5OxRQRxSGEq9fHq39ug",
      "node_name" : "es01",
      "transport_address" : "172.20.0.4:9300",
      "node_attributes" : {
        "ml.machine_memory" : "8312127488",
        "xpack.installed" : "true",
        "transform.node" : "true",
        "ml.max_open_jobs" : "512",
        "ml.max_jvm_size" : "268435456"
      },
      "node_decision" : "no",
      "weight_ranking" : 2,
      "deciders" : [
        {
          "decider" : "same_shard",
          "decision" : "NO",
          "explanation" : "a copy of this shard is already allocated to this node [[.ds-observability-time-series-2022.02.22-000001][0], node[QjZ5OxRQRxSGEq9fHq39ug], [R], s[STARTED], a[id=1Hb2K_nMTHqGP3LqlhDnjQ]]"
        }
      ]
    },
    {
      "node_id" : "g9m0hRGUS1isZhC9CVdhgw",
      "node_name" : "es03",
      "transport_address" : "172.20.0.3:9300",
      "node_attributes" : {
        "ml.machine_memory" : "8312127488",
        "ml.max_open_jobs" : "512",
        "xpack.installed" : "true",
        "ml.max_jvm_size" : "268435456",
        "transform.node" : "true"
      },
      "node_decision" : "no",
      "weight_ranking" : 3,
      "deciders" : [
        {
          "decider" : "same_shard",
          "decision" : "NO",
          "explanation" : "a copy of this shard is already allocated to this node [[.ds-observability-time-series-2022.02.22-000001][0], node[g9m0hRGUS1isZhC9CVdhgw], [P], s[STARTED], a[id=iwUIWrc0S5SSE2R52VK8Mg]]"
        }
      ]
    }
  ]
}
```

## A vous de jouez !

Expliquer le problème que rencontre le cluster Elasticsearch et la solution que vous proposez pour corriger ce comportement.

