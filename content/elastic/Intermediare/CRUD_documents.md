---
title: "Labs - CRUD"
date: 2020-04-29T10:10:47+02:00
draft: false
weight: 30
---

## Description

Quelle que soit la façon dont vous comptez utiliser Elasticsearch, il est essentiel de comprendre comment créer, mettre à jour et supprimer rapidement des documents dans Elasticsearch. Dans cette activité d'apprentissage, vous allez effectuer les tâches suivantes :

* Créer des documents dans Elasticsearch
* Mettre à jour des documents existants dans Elasticsearch
* Supprimer des documents d'un index Elasticsearch

Vous travaillez en tant qu'administrateur Elasticsearch pour une société bancaire. Un récent échec de déploiement et un retour en arrière de votre logiciel bancaire ont désynchronisé certaines actions qui ont été prises sur quelques comptes. Pour corriger rapidement la désynchronisation, on vous demande d'effectuer des opérations CRUD manuelles sur l'index bancaire dans Elasticsearch.

Un compte doit être ajouté avec les données client suivantes :

* Numéro de compte : 1000
* Solde : 65 536
* Prénom : Jean
* Nom de famille : Doe
* Age : 23 ans
* Sexe : masculin Homme
* Adresse : 125 Bear Creek Pkwy
* Employeur : Linux Academy
* Courriel : john@linuxacademy.com
* Ville : Keller
* Etat : TX

Le compte 100 a changé d'adresse et les champs suivants doivent être mis à jour :

* Adresse : 1600 Pennsylvania Ave NW
* Ville : Washington
* Etat : DC

#### Créer Account 1000

```
PUT bank/_doc/1000
{
  "account_number": 1000,
  "balance": 65536,
  "firstname": "John",
  "lastname": "Doe",
  "age": 23,
  "gender": "M",
  "address": "125 Bear Creek Pkwy",
  "employer": "Linux Academy",
  "email": "john@linuxacademy.com",
  "city": "Keller",
  "state": "TX"
}
```

#### Mise à jour Account 100

```
POST bank/_update/100/
{
  "doc": {
    "address": "1600 Pennsylvania Ave NW",
    "city": "Washington",
    "state": "DC"
  }
}
```

#### Supprimer accounts 1 et 10

```
DELETE bank/_doc/1
DELETE bank/_doc/10
```