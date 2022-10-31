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

### Déployer la stack Elasticsearch Kibana dans Docker avec TLS activé

Si les fonctions de sécurité sont activées, vous devez configurer le chiffrement TLS (Transport Layer Security) pour la couche de transport d'Elasticsearch. Bien qu'il soit possible d'utiliser une licence d'essai sans configurer TLS, nous vous conseillons de sécuriser votre pile dès le départ.

Pour obtenir un cluster Elasticsearch et Kibana opérationnel dans Docker avec la sécurité activée, vous pouvez utiliser Docker Compose :

Créez les fichiers de composition et de configuration suivants. Ces fichiers sont également disponibles dans le dépôt elastic/stack-docs sur GitHub.

**instances.yml** identifie les instances pour lesquelles vous devez créer des certificats.
**.env** définit des variables d'environnement pour spécifier la version d'Elasticsearch et l'emplacement où les certificats Elasticsearch seront créés.
**create-certs.yml** est un fichier Docker Compose qui lance un conteneur pour générer les certificats pour Elasticsearch et Kibana.
**elastic-docker-tls.yml** est un fichier Docker Compose qui met en place un cluster Elasticsearch à trois nœuds et une instance Kibana avec TLS activé afin que vous puissiez voir comment les choses fonctionnent. Cette configuration tout-en-un est un moyen pratique de mettre en place votre premier cluster de développement avant de construire un déploiement distribué avec plusieurs hôtes.


Voici l'arborscence attendu :

```
./
├── app
│   ├── artificial_load.sh
│   ├── conf
│   │   └── default.conf
│   └── petclinic.yml
└── elastic-stack
    ├── beats
    │   ├── beats-docker.yml
    │   └── conf
    │       ├── filebeat.yml
    │       ├── heartbeat.yml
    │       └── metricbeat.yml
    ├── create-certs.yml
    ├── data
    │   ├── accounts.json
    │   ├── logs.json
    │   └── shakespeare.json
    ├── elastic-docker-tls.yml
    ├── elastic-fleet-docker-tls.yml
    └── instances.yml
```

**instances.yml** :

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

**.env**:

```
COMPOSE_PROJECT_NAME=es
CERTS_DIR=/usr/share/elasticsearch/config/certificates
VERSION=7.17.0
```

**create-certs.yml**:

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


**elastic-docker-tls.yml**:

```
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:${VERSION}
    container_name: es01
    environment:
      - path.repo=/usr/share/elasticsearch/snapshots/
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - xpack.license.self_generated.type=basic
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
      - path.repo=/usr/share/elasticsearch/snapshots/
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - xpack.license.self_generated.type=basic
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
      - path.repo=/usr/share/elasticsearch/snapshots/
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - xpack.license.self_generated.type=basic
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
    ports:
      - 5601:5601
    environment:
      SERVERNAME: localhost
      ELASTICSEARCH_URL: https://es01:9200
      ELASTICSEARCH_HOSTS: https://es01:9200
      ELASTICSEARCH_USERNAME: $USER_KIBANA_SYSTEM
      ELASTICSEARCH_PASSWORD: $PASSWORD_KIBANA_SYSTEM
      ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES: $CERTS_DIR/ca/ca.crt
      SERVER_SSL_ENABLED: "true"
      SERVER_SSL_KEY: $CERTS_DIR/kib01/kib01.key
      SERVER_SSL_CERTIFICATE: $CERTS_DIR/kib01/kib01.crt
      SERVER_SSL_CERTIFICATEAUTHORITIES: $CERTS_DIR/ca/ca.crt
      XPACK_SECURITY_ENCRYPTIONKEY: "fhjskloppd678ehkdfdlliverpoolfcr"
      XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY: "fhjskloppd678ehkdfdlliverpoolfcr"
    healthcheck:
       test: curl --cacert $CERTS_DIR/ca/ca.crt -s https://localhost/api/status | grep -q 'Looking good' >/dev/null; if [[ $$? == 52 ]]; then echo 0; else echo 1; fi
       retries: 100
       interval: 5s
    volumes:
      - certs:$CERTS_DIR
    networks:
      - elastic
  apm-server:
    image: docker.elastic.co/apm/apm-server:${VERSION}
    container_name: apm-server
    depends_on:
      es01: { condition: service_healthy }
      kib01: { condition: service_healthy }
    cap_add: ["CHOWN", "DAC_OVERRIDE", "SETGID", "SETUID"]
    cap_drop: ["ALL"]
    ports:
    - 8200:8200
    volumes:
       - certs:$CERTS_DIR
    networks:
    - elastic
    command: >
       apm-server -e
         -E apm-server.rum.enabled=true
         -E setup.kibana.host=kib01:5601
         -E setup.template.settings.index.number_of_replicas=1
         -E apm-server.kibana.enabled=true
         -E apm-server.kibana.host=kib01:5601
         -E apm-server.kibana.protocol=https
         -E apm-server.kibana.username=$USER_ELASTIC
         -E apm-server.kibana.password=$PASSWORD_ELASTIC
         -E apm-server.kibana.ssl.enabled=true
         -E apm-server.kibana.ssl.certificate_authorities=["$CERTS_DIR/ca/ca.crt"]
         -E output.elasticsearch.hosts=["https://es01:9200"]
         -E output.elasticsearch.username=$USER_ELASTIC
         -E output.elasticsearch.password=$PASSWORD_ELASTIC
         -E output.elasticsearch.ssl.certificate_authorities=["$CERTS_DIR/ca/ca.crt"]
    healthcheck:
      interval: 10s
      retries: 12
      test: curl --write-out 'HTTP %{http_code}' --fail --silent --output /dev/null http://localhost:8200/

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
    name: elastic
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


Définissez les variables d'authentification dans le fichier **.env**.

```
COMPOSE_PROJECT_NAME=es
CERTS_DIR=/usr/share/elasticsearch/config/certificates
VERSION=7.17.0
### Credentials
USER_APM_SYSTEM=apm_system
PASSWORD_APM_SYSTEM=tE7F2LePRNPgUC4SN0nT
USER_KIBANA_SYSTEM=kibana_system
PASSWORD_KIBANA_SYSTEM=zJE8y8KlV79dv6OCmfpF
USER_KIBANA=kibana
PASSWORD_KIBANA=zJE8y8KlV79dv6OCmfpF
USER_BEATS_SYSTEM=beats_system
PASSWORD_BEATS_SYSTEM=ITklPCO1oTCWQIMdpWFT
USER_ELASTIC=elastic
PASSWORD_ELASTIC=ASf3lYu7xoKMeesYuMPc
```

Utilisez docker-compose pour redémarrer le cluster et Kibana :

```
docker-compose -f elastic-docker-tls.yml stop
docker-compose -f elastic-docker-tls.yml up -d
```

Ouvrez Kibana pour charger les données de l'échantillon et interagir avec le cluster : [Kibana](https://localhost:5601)

![image.png](/elastic-tutorial/images/attachments/prerequis/kibana_connexion.png)

Comme SSL est également activé pour les communications entre Kibana et les navigateurs clients, vous devez accéder à Kibana via le protocole HTTPS.


Ouvrez le menu de droit et cliquez sur "Stack Monitorng"

![image.png](/elastic-tutorial/images/attachments/prerequis/stack_monitoring.png)


Cliquez sur l'option "Or, set up with self monitoring"

![image.png](/elastic-tutorial/images/attachments/prerequis/enable_monitoring.png)


![image.png](/elastic-tutorial/images/attachments/prerequis/turn_on_monitoring.png)

![image.png](/elastic-tutorial/images/attachments/prerequis/monitoring.png)


### Déloyer une application blanche

```
mkdir -p app/conf
vim app/petclinic.yml
```

**petclinic.yml**

```
version: '2.2'

services:
  nginx:
    image: nginx:1.17.3
    container_name: nginx
    labels:
      "co.elastic.logs/module": "nginx"
      "co.elastic.logs/fileset.stdout": "access"
      "co.elastic.logs/fileset.stderr": "error"
      "co.elastic.metrics/module": "nginx"
      "co.elastic.metrics/hosts": "nginx:8081"
      "co.elastic.metrics/metricsets": "stubstatus"
    ports:
      - 8081:8081
    volumes:
      - ./conf/default.conf:/etc/nginx/conf.d/default.conf:ro
    networks:
      - elastic
  petclinic:
    image: docker.io/michaelhyatt/elastic-k8s-o11y-workshop-petclinic:1.25.0
    container_name: petclinic
    labels:
      "co.elastic.metrics/module": "prometheus"
      "co.elastic.metrics/hosts": "$${data.host}:$${data.port}"
      "co.elastic.metrics/metrics_path": "/metrics/prometheus"
      "co.elastic.metrics/period": "1m"
    environment:
      ELASTIC_APM_SERVER_URLS: "http://apm-server:8200"
      ELASTIC_APM_SERVER_URLS_FOR_RUM: "http://localhost:8200"
      ELASTIC_APM_SECRET_TOKEN: ""
      ELASTIC_APM_SERVICE_NAME: "spring-petclinic-monolith"
      ELASTIC_APM_APPLICATION_PACKAGES: "org.springframework.samples"
      ELASTIC_APM_ENABLE_LOG_CORRELATION: "true"
      ELASTIC_APM_CAPTURE_JMX_METRICS: >
        object_name[java.lang:type=GarbageCollector,name=*] attribute[CollectionCount:metric_name=collection_count] attribute[CollectionTime:metric_name=collection_time],
        object_name[java.lang:type=Memory] attribute[HeapMemoryUsage:metric_name=heap]
      JAVA_OPTS: >
        -Xms100m
        -Xmx256m
        -Dspring.profiles.active=mysql
        -Ddatabase=mysql
        -Dspring.datasource.username=root
        -Dspring.datasource.password=petclinic
        -Dspring.datasource.initialization-mode=always
        -Dspring.datasource.url=jdbc:mysql://mysql:3306/petclinic?autoReconnect=true&useSSL=false
        -XX:+StartAttachListener
    ports:
      - 8080
    networks:
      - elastic

  mysql:
    image: mysql:5.7.27
    container_name: mysql
    labels:
      "co.elastic.logs/module": "mysql"
      "co.elastic.metrics/module": "mysql"
      "co.elastic.metrics/hosts": "root:petclinic@tcp($${data.host}:3306)/"
    environment:
      MYSQL_ROOT_PASSWORD: petclinic
      MYSQL_DATABASE: petclinic
    ports:
      - 3306
    networks:
      - elastic

networks:
  elastic:
    name: elastic
    driver: bridge
```

```
vim app/conf/default.conf
```

**default.conf**

```
server {
        listen       8081;
        listen [::]:8081;
        server_name  localhost;
        location /intake {
          if ($request_method = 'OPTIONS') {
            # This is a bit too wide, would be enough to replace it with only APM server url for CORS support.
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'Accept:,Accept-Encoding,Accept-Language:,Cache-Control,Connection,DNT,Pragma,Host,Referer,Upgrade-Insecure-Requests,User-Agent,elastic-apm-traceparent';
            add_header 'Access-Control-Max-Age' 1728000;
            add_header 'Access-Control-Allow-Credentials' 'true';
            add_header 'Content-Type' 'text/plain; charset=utf-8';
            add_header 'Content-Length' 0;
            return 200;
          }
        }
        location / {
          proxy_pass http://petclinic:8080/;
        }
        location = /owners/find {
          return 301 /#$uri;
        }
        location = /vets.html {
          return 301 /#$uri;
        }
        location = /oups {
          return 301 /#$uri;
        }
        location /nginx_status {
         	stub_status on;
        	allow all; # A bit too wide
         	# deny all;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }

```

```
docker-compose -f app/petclinic.yml up -d
```

Accédez à la application blanche en cliquant [ici](http://localhost:8081)

![image.png](/elastic-tutorial/images/attachments/prerequis/petclinic.png)
