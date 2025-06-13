---
title: "Labs - Query niveau I"
date: 2020-04-29T10:10:47+02:00
draft: false
weight: 5
---

## Préparation

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


## API Search

```

### Recherche plusieurs terms (match)

Renvoie les documents qui correspondent à un texte, un nombre, une date ou une valeur booléenne fournis. Le texte fourni est analysé avant de correspondre.


```
GET _search
{
    "query": {
        "match": {
            "text_entry": "to be or not to be"
        }
    }
}
```

### Recherche une phrase en entière (match phrase)

La requête match_phrase analyse le texte et crée une requête de phrase à partir du texte analysé.

```
GET _search
{
    "query": {
        "match_phrase": {
            "text_entry": "to be or not to be"
        }
    }
}
```

### Recherche un term dans un champs spécifique (multi match)

La requête multi_match s'appuie sur la requête match pour permettre les requêtes multi-champs.

```
GET _search
{
    "query": {
        "multi_match": {
            "query": "Crime",
            "fields": ["text_entry", "relatedContent.og:description"]
        }
    }
}
```

### Recherche plusieurs terms dans un champs spécifique (query string)

Renvoie les documents basés sur une chaîne de requête fournie, en utilisant un analyseur syntaxique strict.

```
GET _search
{
    "query": {
        "query_string": {
            "default_field": "text_entry",
            "query": "(romeo AND juliet) OR (mother AND father)"
        }
    }
}
```


### Recherche un term dans un champs spécifique sans utiliser un analyseur de texte (term)

Retourne les documents qui contiennent un terme exact dans un champ fourni.

```
GET _search
{
    "query": {
        "term": {
            "speaker": {
                "value": "ROMEO"
            }
        }
    }
}
```


### Recherche plusieurs terms dans un champs spécifique sans utiliser un analyseur de texte (terms)

Retourne les documents qui contiennent un terme exact dans un champ fourni.

```
GET _search
{
    "query": {
        "terms": {
            "speaker": [
                "ROMEO",
                "JUILIET",
                "HAMLET"
            ]
        }
    }
}
```

### Recherche une valeur entre 40 et 50 dans un champs spécifique (range)

Renvoie les documents qui contiennent des termes compris dans une plage donnée.

```
GET _search
{
    "query": {
        "range": {
            "age": {
                "gte": 40,
                "lt": 50
            }
        }
    }
}
```

### Recherche une valeur wildcard dans un champs spécifique (wildcard)

Renvoie les documents qui contiennent des termes correspondant à un motif de caractère générique.

Un opérateur wildcard est un caractère de remplacement qui correspond à un ou plusieurs caractères. Par exemple, l'opérateur wildcard * correspond à zéro ou plusieurs caractères. Vous pouvez combiner des opérateurs wildcard avec d'autres caractères pour créer un motif wildcard.


```
GET _search
{
    "query": {
        "wildcard": {
            "city.keyword": {
                "value": "*ville"
            }
        }
    }
}
```

### Recherche une valeur avec une regexp dans un champs spécifique (regexp)

Renvoie les documents qui contiennent des termes correspondant à une expression régulière.

```
GET _search
{
    "query": {
        "regexp": {
            "email.keyword": ".*@xurban\\.com"
        }
    }
}
```