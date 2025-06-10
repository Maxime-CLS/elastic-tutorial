---
title: "Installation"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 5
---

## Prérequis

- Machine de 8 CPU, 16Go de RAM et 100Go de disque
- Ubuntu ou Debian ou Docker Desktop for MacOs
- [Docker](https://docs.docker.com/engine/install/ubuntu/) & [Docker-Compose](https://docs.docker.com/compose/install/)
- Exécuter la commande ci-dessous

```
sudo vi /etc/sysctl.conf
```

Ajouter en fin de fichier la ligne suivante : 

```
vm.max_map_count=262144
```

### Vérification de votre environnement de travail

#### Docker

```
docker version
```

Vous devez obtenir une réponse similaire :

```
Client:
 Version:           27.4.0
 API version:       1.47
 Go version:        go1.22.10
 Git commit:        bde2b89
 Built:             Sat Dec  7 10:35:43 2024
 OS/Arch:           darwin/arm64
 Context:           desktop-linux

Server: Docker Desktop 4.37.1 (178610)
 Engine:
  Version:          27.4.0
  API version:      1.47 (minimum version 1.24)
  Go version:       go1.22.10
  Git commit:       92a8393
  Built:            Sat Dec  7 10:38:33 2024
  OS/Arch:          linux/arm64
  Experimental:     false
 containerd:
  Version:          1.7.21
  GitCommit:        472731909fa34bd7bc9c087e4c27943f9835f111
 runc:
  Version:          1.1.13
  GitCommit:        v1.1.13-0-g58aa920
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

#### Docker Compose

```
docker compose version
```

Vous devez obtenir une réponse similaire :

```
Docker Compose version v2.31.0-desktop.2
```

### Déployer la stack Elasticsearch Kibana dans Docker avec TLS activé

Si les fonctions de sécurité sont activées, vous devez configurer le chiffrement TLS (Transport Layer Security) pour la couche de transport d'Elasticsearch. Bien qu'il soit possible d'utiliser une licence d'essai sans configurer TLS, nous vous conseillons de sécuriser votre pile dès le départ.

#### Architecture 

![image.png](/elastic-tutorial/images/attachments/prerequis/architecture.png)

L'architecture de l'Elastic Stack, composée de Elastic Agent, Fleet Server, Elasticsearch, Kibana et le Package Registry, est conçue pour offrir une solution complète de collecte, gestion et analyse des données. Voici une description de chaque composant :

Elastic Agent : Un agent unifié qui collecte des métriques et des logs à partir de diverses sources. Il peut être déployé sur des serveurs, des conteneurs, et des environnements cloud pour centraliser la collecte de données.

Fleet Server : Un composant central qui gère les Elastic Agents. Il distribue les configurations, collecte les informations de statut et coordonne les mises à jour des agents. Fleet Server permet une gestion centralisée et simplifiée des agents.

Elasticsearch : Le moteur de recherche et d'analyse distribué qui stocke et indexe les données collectées. Il permet des recherches rapides et des analyses approfondies grâce à ses capacités de traitement de données en temps réel.

Kibana : L'interface utilisateur pour visualiser et analyser les données stockées dans Elasticsearch. Kibana offre des tableaux de bord interactifs, des outils de visualisation et des capacités de gestion des alertes.

Package Registry : Un dépôt de paquets qui contient des intégrations prêtes à l'emploi pour diverses sources de données. Il permet de télécharger et de mettre à jour facilement les intégrations nécessaires pour collecter des données spécifiques.


#### Description du projet Elastic Platform

Pour obtenir un cluster Elasticsearch et Kibana opérationnel dans Docker avec la sécurité activée, vous pouvez utiliser Docker Compose :

**docker-compose.yml** est un fichier Docker Compose qui met en place un noeud Elasticsearch, une instance Kibana, une instance Fleet Server, une instance de Package Registry et une instance de Elastic Agent avec TLS activé afin que vous puissiez voir comment les choses fonctionnent. Cette configuration tout-en-un est un moyen pratique de mettre en place votre premier déploiement de développement.

Voici l'arborscence :

```
./
├── README.md
├── app
│   ├── artificial_load.sh
│   ├── conf
│   │   └── default.conf
│   └── petclinic.yml
└── elastic-stack
    ├── data
    │   ├── accounts.json
    │   ├── logs.json
    │   └── shakespeare.json
    ├── docker-compose.yml
    ├── elastic-container.sh
    └── kibana.yml
```

Télécharger la projet contenant la plateforme Elastic :

```
git clone https://github.com/Maxime-CLS/elastic-platform.git
```

Démarrer la stack Elastic

```
cd elastic-platform/elastic-stack
chmod +x elastic-container.sh
./elastic-container.sh start
```

```
 ⠿ Container elasticsearch-security-setup  Healthy 7.3s
 ⠿ Container elasticsearch                 Healthy 39.3s
 ⠿ Container kibana                        Healthy 59.3s
 ⠿ Container elastic-agent                 Started 59.7s

Kibana is up. Proceeding

Waiting 40 seconds for Fleet Server setup

Populating Fleet Settings

Populating Synthetics Monitor

READY SET GO!

Browse to https://localhost:5601
Username: elastic
Passphrase: elastic!
```

Vérifiez a présence des volumes

```
docker volume ls
```

```
DRIVER    VOLUME NAME
local     elastic-stack_certs
local     elastic-stack_esdata01
local     elastic-stack_fleetserverdata
local     elastic-stack_kibanadata
```

Vérifier l'état du service Elasticsearch : 

```
curl -k -u 'elastic:elastic' "https://localhost:9200/_cat/health?v"  
```

```
epoch      timestamp cluster        status node.total node.data shards pri relo init unassign unassign.pri pending_tasks max_task_wait_time active_shards_percent
1734951114 10:51:54  docker-cluster yellow          1         1     88  88    0    0       50            0             0                  -                 63.8%
```

Vérifier l'état du service Kibana :

```
curl -k -u 'elastic:elastic' "https://localhost:5601/api/status"  
```

```
{"name":"c4d5cf0d9f00","uuid":"e41c4029-8e86-4348-b498-1fcad701af99","version":{"number":"8.9.0","build_hash":"beb56356c5c037441f89264361302513ff5bd9f8","build_number":64715,"build_snapshot":false,"build_date":"2023-07-19T11:08:05.801Z"},"status":{"overall":{"level":"available","summary":"All services are available"},"core":{"elasticsearch":{"level":"available","summary":"Elasticsearch is available","meta":{"warningNodes":[],"incompatibleNodes":[]}},"savedObjects":{"level":"available","summary":"SavedObjects service has completed migrations and is available","meta":{"migratedIndices":{"migrated":0,"skipped":0,"patched":6}}}},"plugins":{"licensing":{"level":"available","summary":"License fetched"},"banners":{"level":"available","summary":"All dependencies are available"},"customBranding":{"level":"available","summary":"All dependencies are available"},"features":{"level":"available","summary":"All dependencies are available"},"globalSearch":{"level":"available","summary":"All dependencies are available"},"mapsEms":{"level":"available","summary":"All dependencies are available"},"globalSearchProviders":{"level":"available","summary":"All dependencies are available"},"guidedOnboarding":{"level":"available","summary":"All dependencies are available"},"home":{"level":"available","summary":"All dependencies are available"},"console":{"level":"available","summary":"All dependencies are available"},"grokdebugger":{"level":"available","summary":"All dependencies are available"},"management":{"level":"available","summary":"All dependencies are available"},"painlessLab":{"level":"available","summary":"All dependencies are available"},"searchprofiler":{"level":"available","summary":"All dependencies are available"},"advancedSettings":{"level":"available","summary":"All dependencies are available"},"cloudDataMigration":{"level":"available","summary":"All dependencies are available"},"spaces":{"level":"available","summary":"All dependencies are available"},"eventLog":{"level":"available","summary":"All dependencies are available"},"security":{"level":"available","summary":"All dependencies are available"},"cloudLinks":{"level":"available","summary":"All dependencies are available"},"data":{"level":"available","summary":"All dependencies are available"},"encryptedSavedObjects":{"level":"available","summary":"All dependencies are available"},"files":{"level":"available","summary":"All dependencies are available"},"lists":{"level":"available","summary":"All dependencies are available"},"snapshotRestore":{"level":"available","summary":"All dependencies are available"},"telemetry":{"level":"available","summary":"All dependencies are available"},"actions":{"level":"available","summary":"All dependencies are available"},"dataViewEditor":{"level":"available","summary":"All dependencies are available"},"dataViewFieldEditor":{"level":"available","summary":"All dependencies are available"},"ecsDataQualityDashboard":{"level":"available","summary":"All dependencies are available"},"fileUpload":{"level":"available","summary":"All dependencies are available"},"filesManagement":{"level":"available","summary":"All dependencies are available"},"licenseManagement":{"level":"available","summary":"All dependencies are available"},"savedObjects":{"level":"available","summary":"All dependencies are available"},"telemetryManagementSection":{"level":"available","summary":"All dependencies are available"},"unifiedFieldList":{"level":"available","summary":"All dependencies are available"},"ingestPipelines":{"level":"available","summary":"All dependencies are available"},"notifications":{"level":"available","summary":"All dependencies are available"},"savedObjectsTaggingOss":{"level":"available","summary":"All dependencies are available"},"watcher":{"level":"available","summary":"All dependencies are available"},"savedObjectsManagement":{"level":"available","summary":"All dependencies are available"},"savedObjectsTagging":{"level":"available","summary":"All dependencies are available"},"savedSearch":{"level":"available","summary":"All dependencies are available"},"embeddable":{"level":"available","summary":"All dependencies are available"},"globalSearchBar":{"level":"available","summary":"All dependencies are available"},"unifiedSearch":{"level":"available","summary":"All dependencies are available"},"dataViewManagement":{"level":"available","summary":"All dependencies are available"},"imageEmbeddable":{"level":"available","summary":"All dependencies are available"},"navigation":{"level":"available","summary":"All dependencies are available"},"presentationUtil":{"level":"available","summary":"All dependencies are available"},"uiActionsEnhanced":{"level":"available","summary":"All dependencies are available"},"controls":{"level":"available","summary":"All dependencies are available"},"embeddableEnhanced":{"level":"available","summary":"All dependencies are available"},"expressionError":{"level":"available","summary":"All dependencies are available"},"expressionImage":{"level":"available","summary":"All dependencies are available"},"expressionMetric":{"level":"available","summary":"All dependencies are available"},"expressionRepeatImage":{"level":"available","summary":"All dependencies are available"},"expressionRevealImage":{"level":"available","summary":"All dependencies are available"},"expressionShape":{"level":"available","summary":"All dependencies are available"},"graph":{"level":"available","summary":"All dependencies are available"},"kibanaOverview":{"level":"available","summary":"All dependencies are available"},"urlDrilldown":{"level":"available","summary":"All dependencies are available"},"visualizations":{"level":"available","summary":"All dependencies are available"},"dashboard":{"level":"available","summary":"All dependencies are available"},"eventAnnotation":{"level":"available","summary":"All dependencies are available"},"expressionGauge":{"level":"available","summary":"All dependencies are available"},"expressionHeatmap":{"level":"available","summary":"All dependencies are available"},"expressionLegacyMetricVis":{"level":"available","summary":"All dependencies are available"},"expressionMetricVis":{"level":"available","summary":"All dependencies are available"},"expressionPartitionVis":{"level":"available","summary":"All dependencies are available"},"expressionTagcloud":{"level":"available","summary":"All dependencies are available"},"visDefaultEditor":{"level":"available","summary":"All dependencies are available"},"visTypeHeatmap":{"level":"available","summary":"All dependencies are available"},"visTypeMarkdown":{"level":"available","summary":"All dependencies are available"},"visTypeMetric":{"level":"available","summary":"All dependencies are available"},"visTypeTable":{"level":"available","summary":"All dependencies are available"},"visTypeTagcloud":{"level":"available","summary":"All dependencies are available"},"visTypeTimelion":{"level":"available","summary":"All dependencies are available"},"visTypeTimeseries":{"level":"available","summary":"All dependencies are available"},"visTypeVega":{"level":"available","summary":"All dependencies are available"},"visTypeVislib":{"level":"available","summary":"All dependencies are available"},"visTypeXy":{"level":"available","summary":"All dependencies are available"},"dashboardEnhanced":{"level":"available","summary":"All dependencies are available"},"expressionXY":{"level":"available","summary":"All dependencies are available"},"inputControlVis":{"level":"available","summary":"All dependencies are available"},"triggersActionsUi":{"level":"available","summary":"All dependencies are available"},"visTypeGauge":{"level":"available","summary":"All dependencies are available"},"visTypePie":{"level":"available","summary":"All dependencies are available"},"lens":{"level":"available","summary":"All dependencies are available"},"ruleRegistry":{"level":"available","summary":"All dependencies are available"},"stackAlerts":{"level":"available","summary":"All dependencies are available"},"stackConnectors":{"level":"available","summary":"All dependencies are available"},"transform":{"level":"available","summary":"All dependencies are available"},"aiops":{"level":"available","summary":"All dependencies are available"},"cases":{"level":"available","summary":"All dependencies are available"},"discover":{"level":"available","summary":"All dependencies are available"},"maps":{"level":"available","summary":"All dependencies are available"},"dataVisualizer":{"level":"available","summary":"All dependencies are available"},"discoverEnhanced":{"level":"available","summary":"All dependencies are available"},"observabilityShared":{"level":"available","summary":"All dependencies are available"},"reporting":{"level":"available","summary":"All dependencies are available"},"threatIntelligence":{"level":"available","summary":"All dependencies are available"},"timelines":{"level":"available","summary":"All dependencies are available"},"canvas":{"level":"available","summary":"All dependencies are available"},"cloudSecurityPosture":{"level":"available","summary":"All dependencies are available"},"exploratoryView":{"level":"available","summary":"All dependencies are available"},"indexManagement":{"level":"available","summary":"All dependencies are available"},"ml":{"level":"available","summary":"All dependencies are available"},"osquery":{"level":"available","summary":"All dependencies are available"},"reportingExportTypes":{"level":"available","summary":"All dependencies are available"},"sessionView":{"level":"available","summary":"All dependencies are available"},"indexLifecycleManagement":{"level":"available","summary":"All dependencies are available"},"kubernetesSecurity":{"level":"available","summary":"All dependencies are available"},"observability":{"level":"available","summary":"All dependencies are available"},"remoteClusters":{"level":"available","summary":"All dependencies are available"},"rollup":{"level":"available","summary":"All dependencies are available"},"cloudDefend":{"level":"available","summary":"All dependencies are available"},"crossClusterReplication":{"level":"available","summary":"All dependencies are available"},"infra":{"level":"available","summary":"All dependencies are available"},"observabilityOnboarding":{"level":"available","summary":"All dependencies are available"},"synthetics":{"level":"available","summary":"All dependencies are available"},"apm":{"level":"available","summary":"All dependencies are available"},"enterpriseSearch":{"level":"available","summary":"All dependencies are available"},"monitoring":{"level":"available","summary":"All dependencies are available"},"securitySolution":{"level":"available","summary":"All dependencies are available"},"upgradeAssistant":{"level":"available","summary":"All dependencies are available"},"essSecurity":{"level":"available","summary":"All dependencies are available"},"logstash":{"level":"available","summary":"All dependencies are available"},"ux":{"level":"available","summary":"All dependencies are available"},"alerting":{"level":"available","summary":"Alerting is (probably) ready"},"fleet":{"level":"available","summary":"Fleet is available"},"assetManager":{"level":"available","summary":"All dependencies are available"},"bfetch":{"level":"available","summary":"All dependencies are available"},"cloudChatProvider":{"level":"available","summary":"All dependencies are available"},"contentManagement":{"level":"available","summary":"All dependencies are available"},"customIntegrations":{"level":"available","summary":"All dependencies are available"},"esUiShared":{"level":"available","summary":"All dependencies are available"},"expressions":{"level":"available","summary":"All dependencies are available"},"fieldFormats":{"level":"available","summary":"All dependencies are available"},"ftrApis":{"level":"available","summary":"All dependencies are available"},"kibanaReact":{"level":"available","summary":"All dependencies are available"},"kibanaUtils":{"level":"available","summary":"All dependencies are available"},"licenseApiGuard":{"level":"available","summary":"All dependencies are available"},"monitoringCollection":{"level":"available","summary":"All dependencies are available"},"runtimeFields":{"level":"available","summary":"All dependencies are available"},"savedObjectsFinder":{"level":"available","summary":"All dependencies are available"},"screenshotMode":{"level":"available","summary":"All dependencies are available"},"share":{"level":"available","summary":"All dependencies are available"},"textBasedLanguages":{"level":"available","summary":"All dependencies are available"},"translations":{"level":"available","summary":"All dependencies are available"},"unifiedHistogram":{"level":"available","summary":"All dependencies are available"},"urlForwarding":{"level":"available","summary":"All dependencies are available"},"usageCollection":{"level":"available","summary":"All dependencies are available"},"visualizationUiComponents":{"level":"available","summary":"All dependencies are available"},"charts":{"level":"available","summary":"All dependencies are available"},"cloud":{"level":"available","summary":"All dependencies are available"},"dataViews":{"level":"available","summary":"All dependencies are available"},"devTools":{"level":"available","summary":"All dependencies are available"},"inspector":{"level":"available","summary":"All dependencies are available"},"kibanaUsageCollection":{"level":"available","summary":"All dependencies are available"},"newsfeed":{"level":"available","summary":"All dependencies are available"},"telemetryCollectionManager":{"level":"available","summary":"All dependencies are available"},"screenshotting":{"level":"available","summary":"All dependencies are available"},"telemetryCollectionXpack":{"level":"available","summary":"All dependencies are available"},"uiActions":{"level":"available","summary":"All dependencies are available"},"taskManager":{"level":"available","summary":"All dependencies are available"}}},"metrics":{"last_updated":"2023-12-18T16:49:08.209Z","collection_interval_in_millis":5000,"os":{"platform":"linux","platformRelease":"linux-5.10.104-linuxkit","load":{"1m":3.2,"5m":2.94,"15m":1.43},"memory":{"total_in_bytes":8232878080,"free_in_bytes":2566930432,"used_in_bytes":5665947648},"uptime_in_millis":412100,"distro":"Ubuntu","distroRelease":"Ubuntu-20.04","cpu":{"cfs_quota_micros":-1,"cfs_period_micros":100000,"control_group":"/","stat":{"number_of_elapsed_periods":0,"number_of_times_throttled":0,"time_throttled_nanos":0}},"cpuacct":{"control_group":"/","usage_nanos":28507965}},"process":{"memory":{"heap":{"total_in_bytes":321679360,"used_in_bytes":276047768,"size_limit":2197815296},"resident_set_size_in_bytes":438738944},"pid":7,"event_loop_delay":10.360001729729731,"event_loop_delay_histogram":{"min":9.05216,"max":24.985599,"mean":10.360001729729731,"exceeds":0,"stddev":1.1859023333624192,"fromTimestamp":"2023-12-18T16:49:03.209Z","lastUpdatedAt":"2023-12-18T16:49:08.206Z","percentiles":{"50":10.108927,"75":10.207231,"95":11.214847,"99":13.344767}},"event_loop_utilization":{"active":103.03160799999023,"idle":4894.009978000016,"utilization":0.02061852122436791},"uptime_in_millis":201649.885175},"processes":[{"memory":{"heap":{"total_in_bytes":321679360,"used_in_bytes":276047768,"size_limit":2197815296},"resident_set_size_in_bytes":438738944},"pid":7,"event_loop_delay":10.360001729729731,"event_loop_delay_histogram":{"min":9.05216,"max":24.985599,"mean":10.360001729729731,"exceeds":0,"stddev":1.1859023333624192,"fromTimestamp":"2023-12-18T16:49:03.209Z","lastUpdatedAt":"2023-12-18T16:49:08.206Z","percentiles":{"50":10.108927,"75":10.207231,"95":11.214847,"99":13.344767}},"event_loop_utilization":{"active":103.03160799999023,"idle":4894.009978000016,"utilization":0.02061852122436791},"uptime_in_millis":201649.885175}],"response_times":{"avg_in_millis":0,"max_in_millis":0},"concurrent_connections":0,"requests":{"disconnects":0,"total":0,"statusCodes":{},"status_codes":{}},"elasticsearch_client":{"totalActiveSockets":0,"totalIdleSockets":3,"totalQueuedRequests":0}}}%
```

Ouvrez Kibana pour charger les données de l'échantillon et interagir avec le cluster : [Kibana](https://localhost:5601)

![image.png](/elastic-tutorial/images/attachments/prerequis/kibana_connexion.png)

**username :** elastic  
**password :** elastic

Comme SSL est également activé pour les communications entre Kibana et les navigateurs clients, vous devez accéder à Kibana via le protocole HTTPS.

### Déployer une application blanche

```
cd ../app
docker compose -f petclinic.yml up -d
```

Accédez à la application blanche en cliquant [ici](http://localhost:8081)

![image.png](/elastic-tutorial/images/attachments/prerequis/petclinic.png)


Maintenant, ouvrez une nouvelle fenêtre de terminal et exécutez le script suivant pour simuler la charge sur votre serveur NGINX.

```
cd elastic-platform/app/
./artificial_load.sh
```