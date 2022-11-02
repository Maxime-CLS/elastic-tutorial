---
title: "Labs - Query niveau II"
date: 2020-04-29T10:10:47+02:00
draft: false
weight: 10
---

## Préparation

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
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/bank/account/_bulk?pretty' --data-binary @accounts.json
curl -H 'Content-Type: application/x-ndjson' -XPOST 'localhost:9200/shakespeare/doc/_bulk?pretty' --data-binary @shakespeare_6.0.json
```

```
GET _cat/nodes?v

GET _cat/indices?v
```

## Sort

Permet d'ajouter un ou plusieurs tris sur des champs spécifiques. Chaque tri peut également être inversé. Le tri est défini au niveau de chaque champ, avec un nom de champ spécial pour _score pour trier par score, et _doc pour trier par ordre d'index.

### Tri ascendant

```
GET bank/_search
{
    "sort": [
        {
            "account_number": {
                "order": "asc"
            }
        }
    ]
}
```

### Tri descendant

```
GET bank/_search
{
    "sort": [
        {
            "account_number": {
                "order": "desc"
            }
        }
    ]
}
```

### Tri descendant

```
GET bank/_search
{
    "sort": [
        {
            "age": {
                "order": "desc"
            }
        }
    ]
}
```

### Plusieurs tri descendant

```
GET bank/_search
{
    "sort": [
        {
            "age": {
                "order": "desc"
            }
        },
        {
            "balance": {
                "order": "desc"
            }
        }
    ]
}
```

### Tri ascendant

```
GET bank/_search
{
    "sort": [
        {
            "firstname.keyword": {
                "order": "asc"
            }
        }
    ]
}
```