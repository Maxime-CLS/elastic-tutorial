---
title: "Lab - Uptime"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 20
---

Objectif : Dans ce laboratoire, vous explorerez la facilité avec laquelle vous pouvez obtenir des métriques des disponibilités des services avec Heratbeat et les indexer dans Elasticsearch. Vous explorerez également les tableaux de bord Kibana et verrez avec quelle facilité vous pouvez surveiller ces métriques.


### Déployer un agent beats

Ajoutez le service **Heartbeat** à la suite du service **Beats**

```
vim beats/beats-docker.yml
```

**beats-docker.yml**

```
version: '2.2'
services:
  filebeat:
    image: docker.elastic.co/beats/filebeat:${VERSION}
    container_name: filebeat
    restart: unless-stopped
    entrypoint: "filebeat -e -strict.perms=false"
    user: root
    volumes:
    - "./conf/filebeat.yml:/usr/share/filebeat/filebeat.yml:ro"
    - "/var/lib/docker/containers:/var/lib/docker/containers:ro"
    - "/var/run/docker.sock:/var/run/docker.sock:ro"
    - "/var/log:/tmp:ro"
    - certs:$CERTS_DIR
    networks:
    - elastic
  metricbeat:
    image: docker.elastic.co/beats/metricbeat:${VERSION}
    container_name: metricbeat
    restart: unless-stopped
    entrypoint: "metricbeat -e -strict.perms=false"
    user: root
    volumes:
    - "./conf/metricbeat.yml:/usr/share/metricbeat/metricbeat.yml:ro"
    - "/proc:/hostfs/proc:ro"
    - "/sys/fs/cgroup:/hostfs/sys/fs/cgroup:ro"
    - "/:/hostfs:ro"
    - "/var/run/docker.sock:/var/run/docker.sock:ro"
    - certs:$CERTS_DIR
    networks:
    - elastic
  heartbeat:
    image: docker.elastic.co/beats/heartbeat:${VERSION}
    container_name: heartbeat
    restart: unless-stopped
    entrypoint: "heartbeat -e -strict.perms=false"
    user: heartbeat
    volumes:
    - "./conf/heartbeat.yml:/usr/share/heartbeat/heartbeat.yml:ro"
    - "/var/run/docker.sock:/var/run/docker.socki:ro"
    - certs:$CERTS_DIR
    networks:
    - elastic


volumes:
  certs:
    driver: local

networks:
  elastic:
    name: elastic
    driver: bridge
```

```
vim beats/conf/heartbeat.yml
```

```
heartbeat.monitors:
- type: http
  id: nginx-service-status
  name: Nginx Status
  schedule: '@every 10s'
  urls:
    - http://nginx:8081
  timeout: 30s
  check.response:
    status: 200
    body:
      - "Welcome"
- type: http
  id: petclinic-service-status
  name: Petclinic Status
  schedule: '@every 10s'
  urls:
      - http://petclinic:8080/metrics/prometheus
  timeout: 30s
  check.response:
    status: 200
    body:
      - "jvm_memory_committed_bytes"
- type: tcp
  id: mysql-service-status
  name: MySQL Status
  schedule: '@every 10s'
  hosts: ["tcp://mysql:3306"]

processors:
- add_cloud_metadata: ~
- add_docker_metadata: ~

output.elasticsearch:
  hosts: 'https://es01:9200'
  username: "elastic"
  password: "CHANGEME"
  ssl.certificate_authorities: ["/usr/share/elasticsearch/config/certificates/ca/ca.crt"]
```

Changer la valeur du champs **password** avec la valeur du mot de passe générer automatiquement.

```
docker-compose -f beats/beats-docker.yml up -d
```

```
vim app/artificial_load.sh
```

```
#!/bin/bash

COUNTER=0

while [  $COUNTER -lt 10000 ] ;
do
 curl -s --header "X-Forwarded-For: 5.198.223.255" localhost > /dev/null
 curl -s localhost/owners/find > /dev/null
 curl -s localhost/owners?lastName= > /dev/null
 curl -s localhost/owners/1 > /dev/null
 curl -s localhost/owners/4 > /dev/null
 curl -s localhost/owners/6 > /dev/null
 curl -s localhost/owners/8 > /dev/null
 curl -s localhost/vets.html > /dev/null
 curl -s localhost/oups > /dev/null
 let COUNTER=COUNTER+1
done
```

```
chmod +x app/artificial_load.sh
```

Maintenant, ouvrez une nouvelle fenêtre de terminal et exécutez le script suivant pour simuler la charge sur votre serveur NGINX.

```
./app/artificial_load.sh
```

Naviguer dans l'application Petclinic quelque instant pour générer un traffic utilisateur sur un navigateur web.

Lancez Kibana pour commencer à explorer les tableaux de bord qui ont été inclus dans le menu **Observability**

Cliquez sur le bouton **View app**

![image.png](/elastic-tutorial/images/attachments/debutant/observability_uptime.png)

Vous pouvez observer que l'agent beat réalise 3 vérification de disponibilité. Un pour chaque service de l'application.

Cliquez sur le monitor **Nginx Status**

![image.png](/elastic-tutorial/images/attachments/debutant/uptime_monitors.png)

Vous avez maintenant un vue détaillée du service nginx comme les temps de réponses, la disponibilité, etc...

![image.png](/elastic-tutorial/images/attachments/debutant/uptime_nginx_monitor.png)


## Parcours utilisateurs

L'agent Hearbeat permet de simuler un parcours utilisateur via un navigateur. Ce type de vérification permet de valider en tout temps les scénarios d'un site web.


Modifiez la configuration de l'agent Heratbeat

```
vim beats/conf/heartbeat.yml
```

Ajouter les tests de navigation et modifier les URLs indiquer en localhost par l'adresse IP de votre machine. `http://localhost:8081/` en `http://IP_MACHINE:8081/`

```
- type: browser
  id: home2edit-monitor
  name: Home -> Edit Owner journey
  schedule: "@every 10s"
  source:
    inline:
      script: |-
        step("load homepage", async () => {
            await page.goto('http://localhost:8081/');
            await page.waitForRequest(/intake/);
        });
        step("click on find owners", async () => {
            await page.click('#main-navbar > ul > li:nth-child(3) > a');
            await page.waitForRequest(/intake/);
        });
        step("click on find owners button", async () => {
            await page.click('#search-owner-form > div:nth-child(2) > div > button');
            await page.waitForRequest(/intake/);
        });
        step("click on Eduardo Rodriguez", async () => {
            await page.click('#vets > tbody > tr:nth-child(3) > td:nth-child(1) > a');
            await page.waitForRequest(/intake/);
        });
        step("click on edit owner button", async () => {
            await page.click('body > div.container-fluid > div > a:nth-child(3)');
            await page.waitForRequest(/intake/);
        });

- type: browser
  id: home2vets-monitor
  name: Home -> Show Vets journey
  schedule: "@every 10s"
  source:
    inline:
      script: |-
        step("load homepage", async () => {
            await page.goto('http://localhost:8081/');
            await page.waitForRequest(/intake/);
        });
        step("click on vets", async () => {
            await page.click('#main-navbar > ul > li:nth-child(4) > a');
            await page.waitForRequest(/intake/);
        });

- type: browser
  id: home2addpet-monitor
  name: Home -> Add Owner journey
  schedule: "@every 10s"
  source:
    inline:
      script: |-
        step("load homepage", async () => {
            await page.goto('http://localhost:8081/');
            await page.waitForRequest(/intake/);
        });
        step("click on find owners", async () => {
            await page.click('#main-navbar > ul > li:nth-child(3) > a');
            await page.waitForRequest(/intake/);
        });
        step("click on add Owner button", async () => {
            await page.click('body > div.container-fluid > div > a');
            await page.waitForRequest(/intake/);
        });

- type: browser
  id: error-monitor
  name: Home -> Error journey
  schedule: "@every 10s"
  source:
    inline:
      script: |-
        step("load homepage", async () => {
            await page.goto('http://localhost:8081/');
            await page.waitForRequest(/intake/);
        });
        step("click on error", async () => {
            await page.click('#main-navbar > ul > li:nth-child(5) > a');
            await page.waitForRequest(/intake/);
```

Vous pouvez naviguer dans l'interface Kibana Uptime pour visualiser les parcours utilisateurs :

![image.png](/elastic-tutorial/images/attachments/debutant/uptime_browser_monitors.png)

Vous pouvez naviguer le parcours utilisateur **Home -> Show Vets journey - inline** pour visualiser les données de performances ainsi ques des captures d'écran :

![image.png](/elastic-tutorial/images/attachments/debutant/monitor_browser_vets.png)



## A vous de jouez !

On peut constater au travers des autres parcours utilisateurs que l'application n'a pas le comportement attendu. Proposer moi une analyse permettant de comprendre l'origine du problème grace a la solution Elastic.
Et proposer moi une solution pour corriger le dysfonctionnement.




Résumé : Dans ce laboratoire, vous avez exploré la facilité avec laquelle vous pouvez obtenir des mesures des vos services liée à l'application Petclinic avec Heartbeat et les indexer dans Elasticsearch. Vous avez également exploré les tableaux de bord Kibana et vu avec quelle facilité vous pouvez surveiller ces mesures.

