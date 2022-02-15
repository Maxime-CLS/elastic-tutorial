---
title: "Installation"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 5
---

## Prérequis

- Ubuntu ou Debian
- Docker
- Exécuter la commande ci-dessous

```
sudo sysctl -w vm.max_map_count=262144
```

### Exécution dans Docker avec TLS activé

Si les fonctions de sécurité sont activées, vous devez configurer le chiffrement TLS (Transport Layer Security) pour la couche de transport d'Elasticsearch. Bien qu'il soit possible d'utiliser une licence d'essai sans configurer TLS, nous vous conseillons de sécuriser votre pile dès le départ.

Pour obtenir un cluster Elasticsearch et Kibana opérationnel dans Docker avec la sécurité activée, vous pouvez utiliser Docker Compose :

Créez les fichiers de composition et de configuration suivants. Ces fichiers sont également disponibles dans le dépôt elastic/stack-docs sur GitHub.

**instances.yml** identifie les instances pour lesquelles vous devez créer des certificats.
**.env** définit des variables d'environnement pour spécifier la version d'Elasticsearch et l'emplacement où les certificats Elasticsearch seront créés.
**create-certs.yml** est un fichier Docker Compose qui lance un conteneur pour générer les certificats pour Elasticsearch et Kibana.
**elastic-docker-tls.yml** est un fichier Docker Compose qui met en place un cluster Elasticsearch à trois nœuds et une instance Kibana avec TLS activé afin que vous puissiez voir comment les choses fonctionnent. Cette configuration tout-en-un est un moyen pratique de mettre en place votre premier cluster de développement avant de construire un déploiement distribué avec plusieurs hôtes.

instances.yml :

```
instances:
  - name: es01
    dns:
      - es01
      - localhost
    ip:
      - 127.0.0.1

  - name: es02
    dns:
      - es02
      - localhost
    ip:
      - 127.0.0.1

  - name: es03
    dns:
      - es03
      - localhost
    ip:
      - 127.0.0.1

  - name: 'kib01'
    dns:
      - kib01
      - localhost

  - name: 'package-registry'
    dns:
      - package-registry
      - localhost

  - name: 'fleet-server'
    dns:
      - fleet-server
      - localhost
```

.env:

```
COMPOSE_PROJECT_NAME=es
CERTS_DIR=/usr/share/elasticsearch/config/certificates
VERSION=7.17.0
create-certs.yml:
```

```
version: '2.2'

services:
  create_certs:
    image: docker.elastic.co/elasticsearch/elasticsearch:${VERSION}
    container_name: create_certs
    command: >
      bash -c '
        yum install -y -q -e 0 unzip;
        if [[ ! -f /certs/bundle.zip ]]; then
          bin/elasticsearch-certutil cert --silent --pem --in config/certificates/instances.yml -out /certs/bundle.zip;
          unzip /certs/bundle.zip -d /certs;
        fi;
        chown -R 1000:0 /certs
      '
    working_dir: /usr/share/elasticsearch
    volumes:
      - certs:/certs
      - .:/usr/share/elasticsearch/config/certificates
    networks:
      - elastic

volumes:
  certs:
    driver: local

networks:
  elastic:
    driver: bridge
```


elastic-docker-tls.yml:

```
version: '2.2'

services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:${VERSION}
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.license.self_generated.type=trial
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=$CERTS_DIR/es01/es01.key
      - xpack.security.http.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.http.ssl.certificate=$CERTS_DIR/es01/es01.crt
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.security.transport.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.transport.ssl.certificate=$CERTS_DIR/es01/es01.crt
      - xpack.security.transport.ssl.key=$CERTS_DIR/es01/es01.key
      - xpack.security.authc.api_key.enabled=true
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
      - certs:$CERTS_DIR
    ports:
      - 9200:9200
    networks:
      - elastic

    healthcheck:
      test: curl --cacert $CERTS_DIR/ca/ca.crt -s https://localhost:9200 >/dev/null; if [[ $$? == 52 ]]; then echo 0; else echo 1; fi
      interval: 30s
      timeout: 10s
      retries: 5

  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:${VERSION}
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.license.self_generated.type=trial
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=$CERTS_DIR/es02/es02.key
      - xpack.security.http.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.http.ssl.certificate=$CERTS_DIR/es02/es02.crt
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.security.transport.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.transport.ssl.certificate=$CERTS_DIR/es02/es02.crt
      - xpack.security.transport.ssl.key=$CERTS_DIR/es02/es02.key
      - xpack.security.authc.api_key.enabled=true
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
      - certs:$CERTS_DIR
    networks:
      - elastic

  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:${VERSION}
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.license.self_generated.type=trial
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=$CERTS_DIR/es03/es03.key
      - xpack.security.http.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.http.ssl.certificate=$CERTS_DIR/es03/es03.crt
      - xpack.security.transport.ssl.enabled=true
      - xpack.security.transport.ssl.verification_mode=certificate
      - xpack.security.transport.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.transport.ssl.certificate=$CERTS_DIR/es03/es03.crt
      - xpack.security.transport.ssl.key=$CERTS_DIR/es03/es03.key
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
      - certs:$CERTS_DIR
    networks:
      - elastic
  kib01:
    image: docker.elastic.co/kibana/kibana:${VERSION}
    container_name: kib01
    depends_on:
      es01: { condition: service_healthy}
      package-registry: { condition: service_healthy }
    ports:
      - 5601:5601
    environment:
      SERVERNAME: localhost
      ELASTICSEARCH_URL: https://es01:9200
      ELASTICSEARCH_HOSTS: https://es01:9200
      ELASTICSEARCH_USERNAME: kibana_system
      ELASTICSEARCH_PASSWORD: UZuuB5ABGjTBZPez4aDl
      ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES: $CERTS_DIR/ca/ca.crt
      SERVER_SSL_ENABLED: "true"
      SERVER_SSL_KEY: $CERTS_DIR/kib01/kib01.key
      SERVER_SSL_CERTIFICATE: $CERTS_DIR/kib01/kib01.crt
      SERVER_SSL_CERTIFICATEAUTHORITIES: $CERTS_DIR/ca/ca.crt
      XPACK_SECURITY_ENCRYPTIONKEY: "fhjskloppd678ehkdfdlliverpoolfcr"
      XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY: "fhjskloppd678ehkdfdlliverpoolfcr"
      XPACK_FLEET_AGENTS_ELASTICSEARCH_HOST: "https://es01:9200"
      XPACK_FLEET_REGISTRYURL: "https://package-registry"
      NODE_EXTRA_CA_CERTS: $CERTS_DIR/ca/ca.crt
    healthcheck:
       test: curl --cacert $CERTS_DIR/ca/ca.crt -s https://localhost/api/status | grep -q 'Looking good' >/dev/null; if [[ $$? == 52 ]]; then echo 0; else echo 1; fi
       retries: 100
       interval: 5s
    volumes:
      - certs:$CERTS_DIR
    networks:
      - elastic
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

volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local
  certs:
    driver: local

networks:
  elastic:
    driver: bridge

```

1- Générez et appliquez une licence d'essai qui prend en charge Transport Layer Security.

2- Activez Transport Layer Security pour crypter les communications des clients.

3- Activez Transport Layer Security pour crypter les communications internodales.

4- Autorisez l'utilisation de certificats auto-signés en ne demandant pas de vérification du nom d'hôte.


Générez des certificats pour Elasticsearch en faisant apparaître le conteneur create-certs :

```
docker-compose -f create-certs.yml run --rm create_certs
```

Vérifiez a présence des volumes

```
docker volume ls
```

```
DRIVER    VOLUME NAME
local     es_certs
```

Vérifiez la présence des certificats

```
sudo ls -R /var/lib/docker/volumes/es_certs/_data/
```

```
/var/lib/docker/volumes/es_certs/_data/:
bundle.zip  ca	es01  es02  es03  fleet-server	kib01  package-registry

/var/lib/docker/volumes/es_certs/_data/ca:
ca.crt

/var/lib/docker/volumes/es_certs/_data/es01:
es01.crt  es01.key

/var/lib/docker/volumes/es_certs/_data/es02:
es02.crt  es02.key

/var/lib/docker/volumes/es_certs/_data/es03:
es03.crt  es03.key

/var/lib/docker/volumes/es_certs/_data/fleet-server:
fleet-server.crt  fleet-server.key

/var/lib/docker/volumes/es_certs/_data/kib01:
kib01.crt  kib01.key

/var/lib/docker/volumes/es_certs/_data/package-registry:
package-registry.crt  package-registry.key
```

Mettez en place le cluster Elasticsearch à trois nœuds :

```
docker-compose -f elastic-docker-tls.yml up -d
```

À ce stade, Kibana ne peut pas se connecter au cluster Elasticsearch. Vous devez générer un mot de passe pour l'utilisateur intégré kibana_system, mettre à jour le mot de passe ELASTICSEARCH_PASSWORD dans le fichier compose, et redémarrer pour permettre à Kibana de communiquer avec le cluster sécurisé.

Exécutez l'outil elasticsearch-setup-passwords pour générer des mots de passe pour tous les utilisateurs intégrés, y compris l'utilisateur kibana_system. Si vous n'utilisez pas PowerShell sous Windows, supprimez les caractères `\`de fin de ligne et joignez les lignes avant d'exécuter cette commande.

```
docker exec es01 /bin/bash -c "bin/elasticsearch-setup-passwords \
auto --batch --url https://es01:9200"
```

Si la commande tombe en erreur, veuillez relancer la commande quand le noeud Elasticsearch est pret.

```
Changed password for user apm_system
PASSWORD apm_system = cFupG97Ha1vh8Ntdmi8V

Changed password for user kibana_system
PASSWORD kibana_system = im0I9MGIDXHSy5gpUgmm

Changed password for user kibana
PASSWORD kibana = im0I9MGIDXHSy5gpUgmm

Changed password for user logstash_system
PASSWORD logstash_system = 3ZunAd3WpUcp1tKnb3Rq

Changed password for user beats_system
PASSWORD beats_system = yn88diKHJhU5wF7h9Rh9

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = wfliiMix1pep0Gnm6Ah8

Changed password for user elastic
PASSWORD elastic = fLzlyoIb8RsCT639v7HH
```

Prenez note des mots de passe générés. Vous devez configurer le mot de passe de l'utilisateur kibana_system dans le fichier compose pour permettre à Kibana de se connecter à Elasticsearch, et vous aurez besoin du mot de passe du superutilisateur elastic pour vous connecter à Kibana et soumettre des requêtes à Elasticsearch.


Définissez ELASTICSEARCH_PASSWORD dans le fichier elastic-docker-tls.yml avec le mot de passe généré pour l'utilisateur kibana_system.

```
 kib01:
    image: docker.elastic.co/kibana/kibana:${VERSION}
    container_name: kib01
    depends_on: {"es01": {"condition": "service_healthy"}}
    ports:
      - 5601:5601
    environment:
      SERVERNAME: localhost
      ELASTICSEARCH_URL: https://es01:9200
      ELASTICSEARCH_HOSTS: https://es01:9200
      ELASTICSEARCH_USERNAME: kibana_system
      ELASTICSEARCH_PASSWORD: CHANGEME
      ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES: $CERTS_DIR/ca/ca.crt
      SERVER_SSL_ENABLED: "true"
      SERVER_SSL_KEY: $CERTS_DIR/kib01/kib01.key
      SERVER_SSL_CERTIFICATE: $CERTS_DIR/kib01/kib01.crt
    volumes:
      - certs:$CERTS_DIR
    networks:
      - elastic
```

Définissez ELASTICSEARCH_PASSWORD et KIBANA_PASSWORD dans le fichier elastic-docker-tls.yml avec le mot de passe généré pour l'utilisateur elastic.

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

Utilisez docker-compose pour redémarrer le cluster et Kibana :

```
docker-compose stop
docker-compose -f elastic-docker-tls.yml up -d
```

Ouvrez Kibana pour charger les données de l'échantillon et interagir avec le cluster : https://localhost:5601.

![image.png](/images/attachments/prerequis/kibana_connexion.png)

Comme SSL est également activé pour les communications entre Kibana et les navigateurs clients, vous devez accéder à Kibana via le protocole HTTPS.

Vérifiez la connexion entre le serveur Fleet et Kibana :

**Management** -> **Fleet**

![image.png](/images/attachments/prerequis/fleet.png)


Ouvrez le menu de droit et cliquez sur "Stack Monitorng"

![image.png](/images/attachments/prerequis/stack_monitoring.png)


Cliquez sur l'option "Or, set up with self monitoring"

![image.png](/images/attachments/prerequis/enable_monitoring.png)


![image.png](/images/attachments/prerequis/turn_on_monitoring.png)

![image.png](/images/attachments/prerequis/monitoring.png)