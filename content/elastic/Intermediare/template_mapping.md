---
title: "Labs - Index Template et Mapping"
date: 2020-04-29T10:10:47+02:00
draft: false
weight: 18
---

## Description

L'une des fonctionnalités les plus conviviales d'Elasticsearch est le mappage dynamique. Le mappage dynamique est le mécanisme utilisé par Elasticsearch pour détecter les champs et les mettre en correspondance avec un type de données approprié. À l'aide de modèles d'index, nous pouvons définir la structure d'une série d'index pour répondre à des exigences spécifiques ou remplacer et contrôler le comportement du mappage dynamique. Avec le mappage dynamique, nous pouvons sauter le processus de définition explicite de chaque champ possible et de son type de données et indiquer à Elasticsearch comment détecter les types de données souhaités au fur et à mesure qu'ils sont découverts pendant l'indexation. En conséquence, le mappage dynamique nous permet d'être opérationnel très rapidement avec nos données. Dans ce laboratoire pratique, vous allez effectuer les tâches suivantes :

* Créer un modèle d'index
* Définir des mappages de champs explicites
* Définir des mappages de champs dynamiques


Vous travaillez en tant qu'administrateur Elasticsearch pour une société d'analyse de données qui souhaite utiliser votre cluster Elasticsearch existant à 6 nœuds pour analyser quelques ensembles de données. Pour faciliter l'indexation de chaque ensemble de données, vous devez configurer les modèles d'index nécessaires afin que les données soient stockées dans Elasticsearch avec les mappages corrects. Chaque modèle doit avoir 4 shards primaires et 3 shards répliques pour une réplication maximale sur les 4 nœuds de données. Les mappages des modèles requis sont décrits ci-dessous :

| Name      | Index Pattern | Aliases   | Explicit Mapping                                             | Dynamic Mapping                                                                                        |
|-----------|---------------|-----------|--------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| customers | customers-*   | customers | field: year_to_date / type: double                           | name: long_to_integer / match mapping: long / mapping: integers                                        |
| partners  | partners-*    | partners  | field: address / type: text                                  | name: string_to_keyword / match mapping: string / mapping: keyword                                     |
| leads     | leads-*       | leads     | field: address / type: text ; field: estimate / type: double | name: string_to_keyword / match mapping: string / match: lead_* /  unmatch: *_text / mapping: keyword  |

### Créer customers Index Template

Utilisez l'outil de console Kibana pour exécuter ce qui suit :


```
PUT _template/customers
{
  "aliases": {
    "customers": {}
  },
  "index_patterns": ["customers-*"],
  "mappings": {
    "dynamic_templates": [
      {
        "long_to_integer": {
          "match_mapping_type": "long",
          "mapping": {
            "type": "integer"
          }
        }
      }
    ],
    "properties": {
      "year_to_date": {
        "type": "double"
      }
    }
  },
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 3
  }
}
```
### Créer partners Index Template

```
PUT _template/partners
{
  "aliases": {
    "partners": {}
  },
  "index_patterns": ["partners-*"],
  "mappings": {
    "dynamic_templates": [
      {
        "string_to_keyword": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ],
    "properties": {
      "address": {
        "type": "text"
      }
    }
  },
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 3
  }
}
```

## A vous de jouez !

### Créer leads Index Template

A vous de me fournir le template correspondant aux informations dans le tableau.
