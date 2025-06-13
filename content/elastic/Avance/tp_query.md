---
title: "TP : Définir et exécuter des requêtes de recherche dans Elasticsearch"
date: 2020-04-29T10:10:47+02:00
draft: false
weight: 20
---


## Description

Savoir comment collecter, analyser, enrichir et indexer des données dans Elasticsearch est important, mais savoir comment poser des questions précises sur les données est encore plus crucial. Après tout, vous ne pouvez pas épeler "Elasticsearch" sans "recherche" ! Que vous utilisiez Elasticsearch pour la recherche de sites, la recherche de produits, l'analyse opérationnelle ou la veille stratégique, savoir formuler des requêtes de recherche complexes est essentiel pour tirer de la valeur de toutes ces données que vous avez réussi à collecter, analyser, enrichir et indexer. Dans ce laboratoire pratique, vous allez effectuer les tâches suivantes :

* Rechercher un terme spécifique dans un champ
* Appliquer un filtre de recherche pour réduire l'ensemble de données recherchables
* Trier les données résultantes
* Mettre en évidence le terme recherché dans les résultats
* Paginer les résultats de la recherche


Vous travaillez en tant que consultant Elasticsearch et avez été engagé par une université locale qui cherche à mettre en œuvre Elasticsearch pour la recherche littéraire. L'équipe avec laquelle vous travaillez crée une interface utilisateur qui permettra aux étudiants d'effectuer des analyses de recherche sur diverses œuvres littéraires. La configuration de test avec laquelle vous travaillez est un cluster Elasticsearch à 6 nœuds chargé des œuvres complètes de Shakespeare. Pour que l'interface utilisateur affiche les résultats de recherche souhaités, vous devez aider l'équipe à proposer deux requêtes de recherche qui répondent aux exigences.

### Préparation

Les fichiers de données sont stockées dans elastic-stack/data.

```
PUT /shakespeare
{
 "mappings": {
   "properties": {
    "speaker": {"type": "keyword"},
    "play_name": {"type": "keyword"},
    "line_id": {"type": "integer"},
    "speech_number": {"type": "integer"}
   }
 }
}


curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/doc/_bulk?pretty' --data-binary @shakespeare_6.0.json


GET _cat/nodes?v

GET _cat/indices?v

```

### Créer une requête de recherche qui répond aux exigences de la requête

Requête :

* Le type du document doit être une "scène".
* Le champ text_entry doit contenir une forme du mot "london".
* La pièce doit être parmi "Henry VI Part 1", "Henry VI Part 2", ou "Henry VI Part 3".
* La taille du tableau des résultats doit être égale au nombre de résultats.
* Les résultats sont d'abord triés par nom de la pièce dans l'ordre croissant, puis par numéro de ligne dans l'ordre croissant.

