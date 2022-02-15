---
title: "Labs - Query niveau III"
date: 2020-04-29T10:10:47+02:00
draft: false
weight: 15
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

## API Search

```
### Recherche un terms via un boolean
GET _search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "gender.keyword": {
                            "value": "F"
                         }
                    }
                }
            ]
        }
    }
}
```

### Recherche un terms via un boolean et ne pas recherche un term dans les résultats

```
GET _search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "gender.keyword": {
                            "value": "F"
                         }
                    }
                }
            ],
            "must_not": [
                {
                    "term": {
                        "state.keyword": {
                            "value": "RI"
                        }
                    }
                }
            ]
        }
    }
}
```

### Recherche un terms via un boolean, ne pas rechercher un term dans les résultats et de rechercher des résultats avec au moins un des terms

```
GET _search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "gender.keyword": {
                            "value": "F"
                         }
                    }
                }
            ],
            "must_not": [
                {
                    "term": {
                        "state.keyword": {
                            "value": "RI"
                        }
                    }
                }
            ],
            "should": [
                {
                    "term": {
                        "lastname.keyword": {
                            "value": "Meyers"
                        }
                    }
                },
                {
                    "term": {
                        "lastname.keyword": {
                            "value": "Owens"
                        }
                    }
                }
            ],
            "minimum_should_match": 1
        }
    }
}
```


### Recherche un terms via un boolean, ne pas rechercher un term dans les résultats, de rechercher des résultats avec au moins un des terms et un filtre sur la ville

```
GET _search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "gender.keyword": {
                            "_name": "gender",
                            "value": "F"
                         }
                    }
                }
            ],
            "must_not": [
                {
                    "term": {
                        "state.keyword": {
                            "_name": "state",
                            "value": "RI"
                        }
                    }
                }
            ],
            "should": [
                {
                    "term": {
                        "lastname.keyword": {
                            "_name": "lastname_1",
                            "value": "Meyers"
                        }
                    }
                },
                {
                    "term": {
                        "lastname.keyword": {
                            "_name": "lastname_2",
                            "value": "Owens"
                        }
                    }
                }
            ],
            "minimum_should_match": 1,
            "filter": {
                "term": {
                    "city.keyword": "Jacksonburg"
                }
            }
        }
    }
}
```

### Meme requete qu'au dessus avec le meme score

```
GET _search
{
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "gender.keyword": {
                            "_name": "gender",
                            "value": "F"
                         }
                    }
                }
            ],
            "should": [
                {
                    "term": {
                        "lastname.keyword": {
                            "_name": "lastname_1",
                            "value": "Meyers"
                        }
                    }
                }
            ],
            "minimum_should_match": 1,
            "filter": {
                "term": {
                    "city.keyword": "Jacksonburg"
                }
            }
        }
    }
}
```