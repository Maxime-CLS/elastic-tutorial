---
title: "Labs - Reindex"
date: 2020-04-29T10:10:47+02:00
draft: false
weight: 25
---

## Description

Que vous ayez besoin de modifier le mappage d'un index existant ou de prendre un sous-ensemble de données d'un index et de le copier dans un autre, l'API _reindex d'Elasticsearch vous couvre. Avec l'API _reindex, vous pouvez prendre toutes les données ou seulement un sous-ensemble de données d'un index et les copier dans un autre. Dans ce laboratoire pratique, vous aurez l'occasion de faire les exercices suivants :

* Réindexer un sous-ensemble de données d'un index vers un nouvel index.
* Créer un pipeline de nœuds d'ingestion
* Transformer les données pendant le processus de réindexation

Vous travaillez en tant que bibliothécaire de recherche et vous étudiez actuellement les œuvres de Shakespeare, en particulier Roméo et Juliette. Vous avez un cluster Elasticsearch à 6 nœuds et les œuvres complètes de Shakespeare, que vous utilisez pour votre analyse littéraire. Actuellement, les œuvres complètes de Shakespeare sont indexées dans un seul index appelé shakespeare, mais, comme vous vous concentrez actuellement sur la pièce Roméo et Juliette, vous préférez copier cette pièce dans son propre index.

Pour ce faire, vous devrez d'abord créer un nouvel index appelé romeo_and_juliet avec les mêmes mappages de champs que l'index shakespeare. Comme votre cluster Elasticsearch à 3 nœuds ne compte que 4 nœuds de données, vous souhaitez créer l'index romeo_and_juliet avec 4 shards primaires et 3 shards répliques pour une réplication maximale. Une fois l'index romeo_and_juliet créé, vous devrez utiliser l'API _reindex pour copier tous les documents dont le nom de pièce est "Romeo and Juliet" vers l'index romeo_and_juliet.

En plus de copier les données de la pièce Roméo et Juliette dans son propre index, vous souhaitez également modifier les données en cours de route pendant le processus de réindexation. Plus précisément, vous voulez prendre le contenu du champ text_entry et stocker chaque mot délimité par des espaces dans un tableau appelé word_array. En outre, vous voulez ajouter un champ word_count qui est égal au nombre de mots dans le champ word_array. Enfin, comme l'index ne contiendra que les données relatives à la pièce Roméo et Juliette, nous pouvons supprimer le champ play_name. Tout ceci peut être accompli avec un pipeline de nœuds d'acquisition utilisant les processeurs split, script et remove.


#### Préparation des données

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
```

```
curl --location --request POST 'https://localhost:9200/bank/account/_bulk?pretty' \
--header 'Content-Type: application/x-ndjson' \
--header 'Authorization: Basic ZWxhc3RpYzpXc1dnVkFUaUUxSWxsd2tEUXlQbw==' \
 --data-binary @accounts.json

Ecrivez un script de chargement des données nommée load_data_shakespeare.sh

```
#!/bin/bash

input="account.json"
output="bulk.log"
counter=0
max_rows=500
create='{"create": {}}'
bulk_data=$'\n'

echo "Reading logs events from $input..."

```
while read -r log_event
do
  let "counter=counter+1"
  bulk_data+="$create"$'\n'"$log_event"$'\n'
  if [ $counter -eq $max_rows ]
  then
       echo "Indexing $counter documents..."
       bulk_data+=$'\n'
       echo "$bulk_data" | tee temp.json > /dev/null
       curl -XPOST 'https://elastic:nonprodpwd@localhost:9200/account/_bulk' -H 'Content-Type: application/json' --insecure --data-binary @temp.json > "$output"
       rm -rf temp.json
       counter=0
       bulk_data=$'\n'
  fi
done < "$input"

if [ $counter -lt $max_rows ] && [ $counter -gt 0 ]
then
       echo "Indexing $counter documents..."
       bulk_data+=$'\n'
       echo "$bulk_data" | tee temp.json > /dev/null
       curl -XPOST 'https://elastic:nonprodpwd@localhost:9200/account/_bulk' -H 'Content-Type: application/json' --insecure --data-binary @temp.json > "$output"
       rm -rf temp.json
fi
```

Modifier le password d'Elasticsearch.

Exécuter le script.

```
./load_data_account.sh
```

Ecrivez un script de chargement des données nommée load_data_shakespeare.sh

```
#!/bin/bash

input="shakespeare.json"
output="bulk.log"
counter=0
max_rows=500
create='{"create": {}}'
bulk_data=$'\n'

echo "Reading logs events from $input..."

while read -r log_event
do
  let "counter=counter+1"
  bulk_data+="$create"$'\n'"$log_event"$'\n'
  if [ $counter -eq $max_rows ]
  then
       echo "Indexing $counter documents..."
       bulk_data+=$'\n'
       echo "$bulk_data" | tee temp.json > /dev/null
       curl -XPOST 'https://elastic:nonprodpwd@localhost:9200/shakespeare/_bulk' -H 'Content-Type: application/json' --insecure --data-binary @temp.json > "$output"
       rm -rf temp.json
       counter=0
       bulk_data=$'\n'
  fi
done < "$input"

if [ $counter -lt $max_rows ] && [ $counter -gt 0 ]
then
       echo "Indexing $counter documents..."
       bulk_data+=$'\n'
       echo "$bulk_data" | tee temp.json > /dev/null
       curl -XPOST 'https://elastic:nonprodpwd@localhost:9200/shakespeare/_bulk' -H 'Content-Type: application/json' --insecure --data-binary @temp.json > "$output"
       rm -rf temp.json
fi
```

Modifier le password d'Elasticsearch.

Exécuter le script.

```
./load_data_shakespeare.sh
```


#### Créer l'index romeo_and_juiliet

```
PUT romeo_and_juliet
{
  "mappings": {
    "properties": {
      "line_id": {
        "type": "integer"
      },
      "line_number": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "play_name": {
        "type": "keyword"
      },
      "speaker": {
        "type": "keyword"
      },
      "speech_number": {
        "type": "integer"
      },
      "text_entry": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "type": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  },
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 3
  }
}
```

#### Créer le shakespeare-tokenizer ingest node pipeline

```
PUT _ingest/pipeline/shakespeare-tokenizer
{
  "description": "Tokenizes the text_entry field into an array. Adds a word_count field. Removes the play_name field.",
  "processors": [
    {
      "split": {
        "field": "text_entry",
        "separator": "\\s+",
        "target_field": "word_array"
      }
    },
    {
      "script": {
        "lang": "painless",
        "source": "ctx.word_count = ctx.word_array.length"
      }
    },
    {
      "remove": {
        "field": "play_name"
      }
    }
  ]
}
```

#### Reindex le champ play "Romeo and Juliet"

```
POST _reindex
{
  "source": {
    "index": "shakespeare",
    "query": {
      "match": {
        "play_name": "Romeo and Juliet"
      }
    }
  },
  "dest": {
    "index": "romeo_and_juliet",
    "pipeline": "shakespeare-tokenizer"
  }
}
```