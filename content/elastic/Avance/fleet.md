---
title: "Bonus - Fleet & Elastic Agent"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 99
---

### Serveur Fleet

Fleet Server est un composant de la pile Elastic utilisé pour gérer de manière centralisée les agents Elastic. Il est lancé en tant que partie d'un agent Elastic sur un hôte destiné à servir de serveur. Un processus Fleet Server peut prendre en charge de nombreuses connexions d'agents Elastic et sert de plan de contrôle pour la mise à jour des stratégies d'agents, la collecte d'informations d'état et la coordination des actions entre les agents Elastic.

Fleet Server est le mécanisme utilisé par les agents Elastic pour communiquer avec Elasticsearch :

* Lorsqu'une nouvelle politique d'agent est créée, elle est enregistrée dans Elasticsearch.
* Pour s'inscrire à la politique, les agents Elastic envoient une demande à Fleet Server, en utilisant la clé d'inscription générée pour l'authentification.
* Le serveur Fleet reçoit la demande et extrait la politique d'agent d'Elasticsearch, puis l'envoie à tous les agents Elastic inscrits à cette politique.
* L'agent Elastic utilise les informations de configuration de la politique pour collecter et envoyer des données à Elasticsearch.
* Elastic Agent vérifie les mises à jour auprès de Fleet Server, en maintenant une connexion ouverte.
* Lorsqu'une politique est mise à jour, Fleet Server récupère la politique mise à jour dans Elasticsearch et l'envoie aux agents Elastic connectés.

![fleet](https://www.elastic.co/guide/en/fleet/current/images/fleet-server-communication.png)

Ajoutez les deux services ci-dessous dans le fichier **elastic-docker-tls.yml**  :

```
  package-registry:
    image: docker.elastic.co/package-registry/distribution:${VERSION}
    container_name: package-registry
    ports:
      - 443
    environment:
       - EPR_ADDRESS=0.0.0.0:443
       - EPR_TLS_KEY=$CERTS_DIR/package-registry/package-registry.key
       - EPR_TLS_CERT=$CERTS_DIR/package-registry/package-registry.crt
    healthcheck:
      test: curl --cacert $CERTS_DIR/ca/ca.crt -s https://localhost/health >/dev/null; if [[ $$? == 52 ]]; then echo 0; else echo 1; fi
      retries: 100
      interval: 5s
    volumes:
        - certs:$CERTS_DIR
    networks:
       - elastic
  fleet-server:
    image: docker.elastic.co/beats/elastic-agent:${VERSION}
    container_name: fleet-server
    ports:
      - 8220:8220
    healthcheck:
      test: ["CMD-SHELL", "curl -s -k https://localhost:8220/api/status | grep -q 'HEALTHY'"]
      retries: 300
      interval: 1s
    environment:
      FLEET_SERVER_ENABLE: "1"
      ELASTICSEARCH_HOST: "https://es01:9200"
      ELASTICSEARCH_CA: $CERTS_DIR/ca/ca.crt
      ELASTICSEARCH_USERNAME: elastic
      ELASTICSEARCH_PASSWORD: CHANGEME
      FLEET_URL: "https://fleet-server:8220"
      FLEET_SERVER_CERT: $CERTS_DIR/fleet-server/fleet-server.crt
      FLEET_SERVER_CERT_KEY: $CERTS_DIR/fleet-server/fleet-server.key
      KIBANA_FLEET_SETUP: "true"
      KIBANA_HOST: "https://kib01:5601"
      KIBANA_USERNAME: elastic
      KIBANA_PASSWORD: CHANGEME
      KIBANA_CA: $CERTS_DIR/ca/ca.crt
    depends_on:
      es01: { condition: service_healthy }
      kib01: { condition: service_healthy }
    volumes:
      - certs:$CERTS_DIR
    networks:
        - elastic

```

Définissez **ELASTICSEARCH_PASSWORD** et **KIBANA_PASSWORD** dans le fichier **elastic-docker-tls.yml** avec le mot de passe généré pour l'utilisateur elastic.

```
fleet-server:
    image: docker.elastic.co/beats/elastic-agent:${VERSION}
    container_name: fleet-server
    ports:
      - 8220:8220
    healthcheck:
      test: ["CMD-SHELL", "curl -s -k https://localhost:8220/api/status | grep -q 'HEALTHY'"]
      retries: 300
      interval: 1s
    environment:
      FLEET_SERVER_ENABLE: "1"
      ELASTICSEARCH_HOST: "https://es01:9200"
      ELASTICSEARCH_CA: $CERTS_DIR/ca/ca.crt
      ELASTICSEARCH_USERNAME: elastic
      ELASTICSEARCH_PASSWORD: CHANGEME
      FLEET_URL: "https://fleet-server:8220"
      FLEET_SERVER_CERT: $CERTS_DIR/fleet-server/fleet-server.crt
      FLEET_SERVER_CERT_KEY: $CERTS_DIR/fleet-server/fleet-server.key
      KIBANA_FLEET_SETUP: "true"
      KIBANA_HOST: "https://kib01:5601"
      KIBANA_USERNAME: elastic
      KIBANA_PASSWORD: CHANGEME
      KIBANA_CA: $CERTS_DIR/ca/ca.crt
```

```
docker-compose -f elastic-docker-tls.yml up -d
```

Vérifiez la connexion entre le serveur Fleet et Kibana :

**Management** -> **Fleet**

![image.png](/elastic-tutorial/images/attachments/prerequis/fleet.png)


### Serveur APM

Ajoutez le service ELastic Agent dans le fichier **elastic-docker-tls.yml**  : :

```
  elastic-agent:
    image: docker.elastic.co/beats/elastic-agent-complete:${VERSION}
    container_name: elastic-agent
    restart: always
    user: root # note, synthetic browser monitors require this set to `elastic-agent`
    environment:
      FLEET_ENROLL: 1
      FLEET_URL: "https://fleet-server:8220"
      FLEET_TOKEN_POLICY_NAME: "Default policy"
      FLEET_CA: $CERTS_DIR/ca/ca.crt
      KIBANA_FLEET_HOST: https://kib01:5601
      KIBANA_FLEET_USERNAME: elastic
      KIBANA_FLEET_PASSWORD: CHANGEME
      KIBANA_FLEET_CA: $CERTS_DIR/ca/ca.crt
    depends_on:
       es01: { condition: service_healthy }
       fleet-server: { condition: service_healthy }
    volumes:
       - certs:$CERTS_DIR
       - /var/run/docker.sock:/var/run/docker.sock:ro
       - /var/log:/var/log:ro
       - /var/lib/docker/containers:/var/lib/docker/containers:ro
       - /sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro
       - /proc:/hostfs/proc:ro
       - /:/hostfs:ro
    networks:
       - elastic
```

Modifier la configuration par défaut des agents elastic via le serveur fleet sur l'interface Kibana :

**Management** -> **Fleet**

Cliquez sur **Fleet settings**

![image.png](/elastic-tutorial/images/attachments/debutant/fleet_settings.png)

**Fleet Server Hosts**

```
https://fleet-server:8220
```

**Elasticsearch Output Configuration (YAML)**

```
ssl.certificate_authorities: ["/usr/share/elasticsearch/config/certificates/ca/ca.crt"]
```

![image.png](/elastic-tutorial/images/attachments/debutant/fleet_settings_configuration.png)


Définissez KIBANA_FLEET_PASSWORD dans le fichier elastic-docker-tls.yml avec le mot de passe généré pour l'utilisateur elastic.


```
  elastic-agent:
    image: docker.elastic.co/beats/elastic-agent-complete:${VERSION}
    container_name: elastic-agent
    restart: always
    user: root # note, synthetic browser monitors require this set to `elastic-agent`
    environment:
      FLEET_ENROLL: 1
      FLEET_URL: "https://fleet-server:8220"
      FLEET_TOKEN_POLICY_NAME: "Default policy"
      FLEET_CA: $CERTS_DIR/ca/ca.crt
      KIBANA_FLEET_HOST: https://kib01:5601
      KIBANA_FLEET_USERNAME: elastic
      KIBANA_FLEET_PASSWORD: CHANGEME
      KIBANA_FLEET_CA: $CERTS_DIR/ca/ca.crt
    depends_on:
       es01: { condition: service_healthy }
       fleet-server: { condition: service_healthy }
    volumes:
       - certs:$CERTS_DIR
       - /var/run/docker.sock:/var/run/docker.sock:ro
       - /var/log:/var/log:ro
       - /var/lib/docker/containers:/var/lib/docker/containers:ro
       - /sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro
       - /proc:/hostfs/proc:ro
       - /:/hostfs:ro
    networks:
       - elastic
```

Après quelques minutes vous devez voir apparaitre l'agent elastic sur l'interface Kibana.


![image.png](/elastic-tutorial/images/attachments/debutant/fleet_agent_elastic.png)


### Intégration du module APM

