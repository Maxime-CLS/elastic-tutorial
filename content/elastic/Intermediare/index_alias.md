---
title: "Labs - Index et ALias"
date: 2020-04-29T10:10:47+02:00
draft: false
weight: 10
---

## Description

Dans Elasticsearch, les données que nous indexons sont stockées dans un index. Le mot "index" est utilisé à la fois comme un verbe et un nom. Essentiellement, nous effectuons une opération d'indexation afin de stocker des données dans un index. Cependant, avant de pouvoir le faire, nous devons d'abord comprendre la structure d'un index et comment en définir un pour répondre à des besoins spécifiques. Dans ce laboratoire pratique, vous allez définir des index Elasticsearch en effectuant les tâches suivantes :

* Configurer le nombre de shards primaires d'un index.
* Configurer le nombre de tiroirs de réplique d'un index.
* Attribuer les tiroirs d'un index aux nœuds chauds.
* Allouer les tessons d'un index aux nœuds chauds.
* Associer des index avec des alias.

Vous travaillez en tant qu'administrateur système pour une entreprise qui souhaite utiliser Elasticsearch avec Kibana pour stocker et analyser certaines données de journal. On vous demande de préparer le cluster Elasticsearch pour les données de journal en créant quelques index. Les journaux sont considérés comme des données chronologiques, et nous nous intéressons généralement aux journaux les plus récents. Nous devons nous assurer que les données dont nous nous soucions le plus sont allouées à nos nœuds les plus rapides, ou "chauds". Les données dont nous nous soucions moins peuvent être attribuées à des nœuds "chauds" plus lents, qui ne seront pas indexés ou recherchés aussi souvent. Nous devons également être en mesure de rechercher les données en utilisant des alias tels que "this_week", "yesterday" ou "today". Les alias facilitent la recherche des données qui vous intéressent car vous n'avez pas besoin de connaître les noms d'index spécifiques.

Vous disposez d'un cluster Elasticsearch préconfiguré à 6 nœuds avec Kibana déjà configuré et en cours d'exécution sur le nœud coordinator-1. Vous devrez utiliser l'outil de console de Kibana pour interagir avec les API d'Elasticsearch afin de créer les index suivants :

| Name            | Aliases                      | Primaries | Replicas | Allocation |
|-----------------|------------------------------|-----------|----------|------------|
| tuto-logs-2020-01-05 | logs / this_week             | 2         | 1        | warm       |
| tuto-logs-2020-01-06 | logs / this_week             | 2         | 1        | warm       |
| tuto-logs-2020-01-07 | logs / this_week             | 2         | 1        | warm       |
| tuto-logs-2020-01-08 | logs / this_week             | 2         | 1        | warm       |
| tuto-logs-2020-01-09 | logs / this_week             | 2         | 1        | warm       |
| tuto-logs-2020-01-10 | logs / this_week / yesterday | 2         | 1        | hot        |
| tuto-logs-2020-01-11 | logs / this_week / today     | 2         | 1        | hot        |



### Création des index

Utilisez l'outil de console Kibana pour exécuter ce qui suit :

#### Créer l'index tuto-logs-2020-01-05
```
PUT tuto-logs-2020-01-05
{
  "aliases": {
    "logs": {},
    "this_week": {}
  },
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1,
    "index.routing.allocation.require.temp": "warm"
  }
}
```

#### Créer l'index tuto-logs-2020-01-06
```
PUT tuto-logs-2020-01-06
{
  "aliases": {
    "logs": {},
    "this_week": {}
  },
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1,
    "index.routing.allocation.require.temp": "warm"
  }
}
```

#### Créer l'index tuto-logs-2020-01-07
```
PUT tuto-logs-2020-01-07
{
  "aliases": {
    "logs": {},
    "this_week": {}
  },
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1,
    "index.routing.allocation.require.temp": "warm"
  }
}
```

#### Créer l'index tuto-logs-2020-01-08
```
PUT tuto-logs-2020-01-08
{
  "aliases": {
    "logs": {},
    "this_week": {}
  },
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1,
    "index.routing.allocation.require.temp": "warm"
  }
}
```

#### Créer l'index tuto-logs-2020-01-09
```
PUT tuto-logs-2020-01-09
{
  "aliases": {
    "logs": {},
    "this_week": {}
  },
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1,
    "index.routing.allocation.require.temp": "warm"
  }
}
```

#### Créer l'index tuto-logs-2020-01-10
```
PUT tuto-logs-2020-01-10
{
  "aliases": {
    "logs": {},
    "this_week": {},
    "yesterday": {}
  },
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1,
    "index.routing.allocation.require.temp": "hot"
  }
}
```

## A vous de jouez !

#### Créer l'index tuto-logs-2020-01-11

A vous de me fournir la configuration de l'index correspondant aux informations dans le tableau.


