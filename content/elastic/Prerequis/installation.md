---
title: "Installation"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 5
---

## Prérequis

- Ubuntu ou Debian
- Docker & Docker-Compose
- Exécuter la commande ci-dessous

```
sudo sysctl -w vm.max_map_count=262144
```

### Vérification de votre environnement de travail

#### Docker

```
docker version
```

Vous devez obtenir une réponse similaire :

```
Client: Docker Engine - Community
 Version:           24.0.2
 API version:       1.43
 Go version:        go1.20.4
 Git commit:        cb74dfc
 Built:             Thu May 25 21:52:13 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          24.0.2
  API version:      1.43 (minimum version 1.12)
  Go version:       go1.20.4
  Git commit:       659604f
  Built:            Thu May 25 21:52:13 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.21
  GitCommit:        3dce8eb055cbb6872793272b4f20ed16117344f8
 runc:
  Version:          1.1.7
  GitCommit:        v1.1.7-0-g860f061
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

Si vous avez un retour vous indiquant que la commande docker n'est pas reconnu, veuillez l'installer.

```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

Si vous rencontrez des difficultés à utiliser la commande docker sans le sudo, veuillez ajouter cette configruation :

```
sudo groupadd docker
sudo usermod -aG docker $USER
docker run hello-world
```


### Déployer la stack Elasticsearch Kibana dans Docker avec TLS activé

Si les fonctions de sécurité sont activées, vous devez configurer le chiffrement TLS (Transport Layer Security) pour la couche de transport d'Elasticsearch. Bien qu'il soit possible d'utiliser une licence d'essai sans configurer TLS, nous vous conseillons de sécuriser votre pile dès le départ.

Pour obtenir un cluster Elasticsearch et Kibana opérationnel dans Docker avec la sécurité activée, vous pouvez utiliser Docker Compose :


**instances.yml** identifie les instances pour lesquelles vous devez créer des certificats.
**elastic-docker-tls.yml** est un fichier Docker Compose qui met en place un cluster Elasticsearch à trois nœuds et une instance Kibana avec TLS activé afin que vous puissiez voir comment les choses fonctionnent. Cette configuration tout-en-un est un moyen pratique de mettre en place votre premier cluster de développement avant de construire un déploiement distribué avec plusieurs hôtes.
**beats-docker.yml** est un fichier Docker Compose qui met en place les agents de collecte avec TLS activé afin que vous puissiez voir comment les choses fonctionnent.

Voici l'arborscence :

```
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
    ├── data
    │   ├── accounts.json
    │   ├── logs.json
    │   └── shakespeare.json
    ├── elastic-docker-tls.yml
    └── instances.yml

```

Télécharger la projet contenant la plateforme Elastic :

```
git clone https://github.com/Maxime-CLS/elastic-platform.git
```

Démarrer la stack Elastic

```
cd elastic-platform/elastic-stack
docker compose -f elastic-docker-tls.yml up -d
```

Vérifiez a présence des volumes

```
docker volume ls
```

```
DRIVER    VOLUME NAME
local     es_certs
local     es_esdata01
local     es_esdata02
local     es_esdata03
local     es_kibanadata
```

Vérifiez la présence des certificats

```
sudo ls -R /var/lib/docker/volumes/es_certs/_data/
```

```
/var/lib/docker/volumes/es_certs/_data/:
ca  ca.zip  certs.zip  es01  es02  es03  fleet-server  instances.yml  kibana  package-registry

/var/lib/docker/volumes/es_certs/_data/ca:
ca.crt	ca.key

/var/lib/docker/volumes/es_certs/_data/es01:
es01.crt  es01.key

/var/lib/docker/volumes/es_certs/_data/es02:
es02.crt  es02.key

/var/lib/docker/volumes/es_certs/_data/es03:
es03.crt  es03.key

/var/lib/docker/volumes/es_certs/_data/fleet-server:
fleet-server.crt  fleet-server.key

/var/lib/docker/volumes/es_certs/_data/kibana:
kibana.crt  kibana.key

/var/lib/docker/volumes/es_certs/_data/package-registry:
package-registry.crt  package-registry.key
```

Ouvrez Kibana pour charger les données de l'échantillon et interagir avec le cluster : [Kibana](https://localhost:5601)

![image.png](/elastic-tutorial/images/attachments/prerequis/kibana_connexion.png)

**username :** elastic  
**password :** elastic

Comme SSL est également activé pour les communications entre Kibana et les navigateurs clients, vous devez accéder à Kibana via le protocole HTTPS.


Ouvrez le menu de droit et cliquez sur "Stack Monitorng"

![image.png](/elastic-tutorial/images/attachments/prerequis/stack_monitoring.png)


Cliquez sur l'option "Or, set up with self monitoring"

![image.png](/elastic-tutorial/images/attachments/prerequis/enable_monitoring.png)


![image.png](/elastic-tutorial/images/attachments/prerequis/turn_on_monitoring.png)

![image.png](/elastic-tutorial/images/attachments/prerequis/monitoring.png)


### Déployer une application blanche

```
cd ../app
docker compose -f petclinic.yml up -d
```

Accédez à la application blanche en cliquant [ici](http://localhost:8081)

![image.png](/elastic-tutorial/images/attachments/prerequis/petclinic.png)

### Déployer les agents de collectes

```
cd ../elastic-stack
docker compose -f beats/beats-docker.yml up -d
```


### Labs Avancés

#### Télécharger des données d'exemples

```
git clone https://github.com/Maxime-CLS/elastic-data-tutorial.git
```
