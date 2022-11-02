---
title: "Lab - Habilitation"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 5
---

### Roles Kibana

Les rôles sont un ensemble de privilèges qui vous permettent d'effectuer des actions dans Kibana et Elasticsearch. Les utilisateurs ne reçoivent pas directement de privilèges, mais se voient plutôt attribuer un ou plusieurs rôles qui décrivent le niveau d'accès souhaité. Lorsque vous attribuez plusieurs rôles à un utilisateur, ce dernier reçoit une union des privilèges des rôles. Cela signifie que vous ne pouvez pas réduire les privilèges d'un utilisateur en lui attribuant un rôle supplémentaire. Vous devez plutôt supprimer ou modifier l'un de ses rôles existants.

![image.png](/elastic-tutorial/images/attachments/elementaire/role-based-access-control.png)

#### User Observability

Vous allez créer un role pour les utilisateurs qui doivent consulter les différentes informations de leurs applications sur le menu **Observability**.
Cette création sera faite via API d'Elasticsearch. Pour cela vous allez ouvrir un dans le menu Kibana Management -> DevTool.


```
POST /_security/role/read_observability
{
  "cluster": [],
  "indices": [
    {
      "names": [
        "metricbeat-*",
        "filebeat-*",
        "traces-apm*,apm-*,logs-apm*,apm-*,metrics-apm*,apm-*"
      ],
      "privileges": [
        "read"
      ],
      "allow_restricted_indices": false
    }
  ],
  "applications": [
    {
      "application": "kibana-.kibana",
      "privileges": [
        "feature_logs.read",
        "feature_infrastructure.read",
        "feature_apm.read",
        "feature_uptime.read",
        "feature_observabilityCases.read"
      ],
      "resources": [
        "space:default"
      ]
    }
  ],
  "run_as": [],
  "metadata": {},
  "transient_metadata": {
    "enabled": true
  }
}
```

```
{
  "role" : {
    "created" : true
  }
}
```

Vérifions la configuration du role :

```
GET /_security/role/read_observability
```

```
{
  "read_observability" : {
    "cluster" : [ ],
    "indices" : [
      {
        "names" : [
          "metricbeat-*",
          "filebeat-*",
          "traces-apm*,apm-*,logs-apm*,apm-*,metrics-apm*,apm-*"
        ],
        "privileges" : [
          "read"
        ],
        "allow_restricted_indices" : false
      }
    ],
    "applications" : [
      {
        "application" : "kibana-.kibana",
        "privileges" : [
          "feature_logs.read",
          "feature_infrastructure.read",
          "feature_apm.read",
          "feature_uptime.read",
          "feature_observabilityCases.read"
        ],
        "resources" : [
          "space:default"
        ]
      }
    ],
    "run_as" : [ ],
    "metadata" : { },
    "transient_metadata" : {
      "enabled" : true
    }
  }
}
```


Ensuite vous allez définir un utilisateur qui aura l'habilitation de lire les données d'observabilité grace au role précédament crée.


```
POST /_security/user/user_observability
{
  "username": "user_observability",
  "password": "test1234",
  "roles": [
    "read_observability"
  ],
  "full_name": "",
  "email": "",
  "metadata": {},
  "enabled": true
}
```

```
{
  "created" : true
}

```

Vérifions la configuration du role :

```
GET /_security/user/user_observability
```

```
{
  "user_observability" : {
    "username" : "user_observability",
    "roles" : [
      "read_observability"
    ],
    "full_name" : "",
    "email" : "",
    "metadata" : { },
    "enabled" : true
  }
}
```

Maintenant ouvrez une page de votre navigateur privé pour vous connectez à Kibana avec cet utilisateur : [Accerder à Kibana](https://localhost:5601)

![image.png](/elastic-tutorial/images/attachments/elementaire/user_authentification.png)

Sélectionnez dans le menu **Observability**

![image.png](/elastic-tutorial/images/attachments/elementaire/user_menu.png)

Puis observer que l'utilisateur ne voit pas les données de type **APM** et **Uptime**.

![image.png](/elastic-tutorial/images/attachments/elementaire/bug_user_habilitation.png)

## A vous de jouez !

A vous de corriger les habilitations de l'utilisateur pour la lecture de données manquantes, voici quelque lien utile pour votre problème :

- https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-put-role.html
- https://www.elastic.co/guide/en/kibana/current/kibana-role-management.html
- https://www.elastic.co/guide/en/elasticsearch/reference/8.0/security-privileges.html#privileges-list-indices


---

#### Administrator Observability

Vous allez créer un role pour les administrateurs qui doivent consulter/modifier/supprimer les différentes informations de leurs applications sur le menu **Observability**.
Cette création sera faite via API d'Elasticsearch. Pour cela vous allez ouvrir un dans le menu Kibana Management -> DevTool.


Corriger l'erreur dans le role administrateur que vous avez trouver sur le role user, puis exécuter cette requete :

```
POST /_security/role/admin_observability
{
  "cluster": [
    "create_snapshot",
    "manage_ccr",
    "manage_ilm",
    "manage_saml",
    "manage_watcher",
    "monitor",
    "read_ilm"
  ],
  "indices": [
    {
      "names": [
        "metricbeat-*",
        "filebeat-*",
        "traces-apm*,apm-*,logs-apm*,apm-*,metrics-apm*,apm-*"
      ],
      "privileges": [
        "read",
        "monitor",
        "create",
        "write",
        "delete",
        "delete_index"
      ],
      "allow_restricted_indices": false
    }
  ],
  "applications": [
    {
      "application": "kibana-.kibana",
      "privileges": [
        "feature_logs.all",
        "feature_infrastructure.all",
        "feature_apm.all",
        "feature_uptime.all",
        "feature_observabilityCases.all",
        "feature_dev_tools.all",
        "feature_indexPatterns.read"
      ],
      "resources": [
        "space:default"
      ]
    }
  ],
  "run_as": [],
  "metadata": {},
  "transient_metadata": {
    "enabled": true
  }
}
```

```
{
  "role" : {
    "created" : true
  }
}
```

Vérifions la configuration du role :

```
GET /_security/role/admin_observability
```

```
{
  "admin_observability" : {
    "cluster" : [
      "create_snapshot",
      "manage_ccr",
      "manage_ilm",
      "manage_saml",
      "manage_watcher",
      "monitor",
      "read_ilm"
    ],
    "indices" : [
      {
        "names" : [
          "metricbeat-*",
          "filebeat-*",
          "uptime-*",
          "apm-*",
          "heartbeat-*"
        ],
        "privileges" : [
          "read",
          "monitor",
          "create",
          "write",
          "delete",
          "delete_index"
        ],
        "allow_restricted_indices" : false
      }
    ],
    "applications" : [
      {
        "application" : "kibana-.kibana",
        "privileges" : [
          "feature_logs.all",
          "feature_infrastructure.all",
          "feature_apm.all",
          "feature_uptime.all",
          "feature_observabilityCases.all",
          "feature_dev_tools.all",
          "feature_indexPatterns.read"
        ],
        "resources" : [
          "space:default"
        ]
      }
    ],
    "run_as" : [ ],
    "metadata" : { },
    "transient_metadata" : {
      "enabled" : true
    }
  }
}
```


Ensuite vous allez définir un utilisateur qui aura l'habilitation de lire les données d'observabilité grace au role précédament crée.


```
POST /_security/user/admin_observability
{
  "password" : "admin_password",
  "roles": [
    "admin_observability"
  ],
  "full_name": "",
  "email": "",
  "metadata": {},
  "enabled": true
}
```

```
{
  "created" : true
}

```

Vérifions la configuration du role :

```
GET /_security/user/user_observability
```

```
{
  "user_observability" : {
    "username" : "user_observability",
    "roles" : [
      "read_observability"
    ],
    "full_name" : "",
    "email" : "",
    "metadata" : { },
    "enabled" : true
  }
}
```

Maintenant ouvrez une page de votre navigateur privé pour vous connectez à Kibana avec cet utilisateur : [Accerder à Kibana](https://localhost:5601)

![image.png](/elastic-tutorial/images/attachments/elementaire/admin_login.png)

Sélectionnez dans le menu **Observability**

![image.png](/elastic-tutorial/images/attachments/elementaire/user_menu.png)

Puis observer que l'utilisateur ne voit pas les données de type **APM** et **Uptime**.

![image.png](/elastic-tutorial/images/attachments/elementaire/admin_observability.png)


```
GET apm-*/_search
GET metricbeat-*/_search
GET filebeat-*/_search
GET heartbeat-*/_search
```

Maintenant l'administrateur souhaite créer un index dans Elasticsearch afin de regrouper les données en un point d'accès

```
PUT my-index-observability
```

```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "security_exception",
        "reason" : "action [indices:admin/create] is unauthorized for user [admin_observability] with roles [admin_observability], this action is granted by the index privileges [create_index,manage,all]"
      }
    ],
    "type" : "security_exception",
    "reason" : "action [indices:admin/create] is unauthorized for user [admin_observability] with roles [admin_observability], this action is granted by the index privileges [create_index,manage,all]"
  },
  "status" : 403
}
```

## A vous de jouez !

A vous de corriger les habilitations de l'administrateur pour la création d'un index, voici quelque lien utile pour votre problème :

- https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-put-role.html
- https://www.elastic.co/guide/en/kibana/current/kibana-role-management.html
- https://www.elastic.co/guide/en/elasticsearch/reference/8.0/security-privileges.html#privileges-list-indices


Résumé : Dans ce laboratoire, vous avez exploré la facilité avec laquelle vous pouvez créer et mettre à jour des roles pour vos utilisateurs. Vous avez également exploré les différentes habilitations possibles sur Elasticsearch.
