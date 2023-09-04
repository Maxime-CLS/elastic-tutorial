---
title: "Lab - Logs"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 5
---

Objectif : Dans ce laboratoire, vous explorerez la facilité avec laquelle vous pouvez lire les fichiers journaux NGINX avec Filebeat et les indexer dans Elasticsearch. Vous explorerez également les tableaux de bord Kibana et verrez avec quelle facilité vous pouvez surveiller les journaux NGINX.


Voici le fichier de configuration de l'agent de collecte pour les données de type logs.

Cette configuration va nous permettre :
- Collecter les logs de la machine Linux
- Collecter les logs de containers Docker
- Contextualiser les données collecter au travers de metadata
- Envoyer les données à un cluster Elasticsearch
- Provisionner des tableaux de bords
- Provisionner des paramètres de configuration des données de type logs

**filebeat.yml**

```
######################## Filebeat Configuration ############################

#==========================  Modules configuration ============================
filebeat.modules:

#------------------------------- System Module -------------------------------
- module: system
  # Syslog
  syslog:
    enabled: true

  # Authorization logs
  auth:
    enabled: true

#=========================== Filebeat inputs =============================

# List of inputs to fetch data.
filebeat.inputs:

#------------------------------ Log input --------------------------------
- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  # To fetch all ".log" files from a specific level of subdirectories
  # /var/log/*/*.log can be used.
  # For each file found under this path, a harvester is started.
  # Make sure not file is defined twice as this can lead to unexpected behaviour.
  paths:
    - "/var/lib/docker/containers/*/*-json.log"
    - '/tmp/*.log'

#========================== Filebeat autodiscover ==============================

# Autodiscover allows you to detect changes in the system and spawn new modules
# or inputs as they happen.

filebeat.autodiscover:
  # List of enabled autodiscover providers
  providers:
    - type: docker
      hints.enabled: true

#================================ Processors ===================================

#
# The following example enriches each event with docker metadata, it matches
# given fields to an existing container id and adds info from that container:
#
processors:
- add_docker_metadata: ~
- add_locale:
        format: offset
- add_host_metadata:
        netinfo.enabled: true


#================================ Outputs ======================================

# Configure what output to use when sending the data collected by the beat.

#-------------------------- Elasticsearch output -------------------------------
output.elasticsearch:
  # Boolean flag to enable or disable the output module.
  enabled: true

  # Array of hosts to connect to.
  # Scheme and port can be left out and will be set to the default (http and 9200)
  # In case you specify and additional path, the scheme is required: http://localhost:9200/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:9200
  hosts: ["es01:9200"]

  # Set gzip compression level.
  #compression_level: 0

  # Optional protocol and basic auth credentials.
  protocol: "https"
  username: "elastic"
  password: "CHANGEME"

  # Use SSL settings for HTTPS.
  ssl.enabled: true

  # SSL configuration. By default is off.
  # List of root certificates for HTTPS server verifications
  ssl.certificate_authorities: ["/usr/share/elasticsearch/config/certificates/ca/ca.crt"]

#============================== Dashboards =====================================
# These settings control loading the sample dashboards to the Kibana index. Loading
# the dashboards are disabled by default and can be enabled either by setting the
# options here, or by using the `-setup` CLI flag or the `setup` command.
setup.dashboards.enabled: true

#============================== Template =====================================

# Template name. By default the template name is "filebeat-%{[beat.version]}"
# The template name and pattern has to be set in case the elasticsearch index pattern is modified.
setup.template.name: "logs-%{[beat.version]}"

# Template pattern. By default the template pattern is "-%{[beat.version]}-*" to apply to the default index settings.
# The first part is the version of the beat and then -* is used to match all daily indices.
# The template name and pattern has to be set in case the elasticsearch index pattern is modified.
setup.template.pattern: "logs-%{[beat.version]}-*"

# Path to fields.yml file to generate the template
#setup.template.fields: "${path.config}/fields.yml"

# Overwrite existing template
#setup.template.overwrite: false

# Elasticsearch template settings
setup.template.settings:

  # A dictionary of settings to place into the settings.index dictionary
  # of the Elasticsearch template. For more details, please check
  # https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html
  #index:
    number_of_shards: 1
    #codec: best_compression
    #number_of_routing_shards: 30

  # A dictionary of settings for the _source field. For more details, please check
  # https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-source-field.html
  #_source:
    #enabled: false

#============================== Kibana =====================================

# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  host: "kib01:5601"

  # Optional protocol and basic auth credentials.
  protocol: "https"
  username: "elastic"
  password: "CHANGEME"

  # Use SSL settings for HTTPS. Default is true.
  ssl.enabled: true
  ssl.certificate_authorities: ["/usr/share/elasticsearch/config/certificates/ca/ca.crt"]

#================================ Logging ======================================
# There are four options for the log output: file, stderr, syslog, eventlog
# The file output is the default.

# Sets log level. The default log level is info.
# Available log levels are: error, warning, info, debug
logging.level: info

# Send all logging output to syslog. The default is false.
logging.to_syslog: true

# Send all logging output to Windows Event Logs. The default is false.
logging.to_eventlog: true

# If enabled, filebeat periodically logs its internal metrics that have changed
# in the last period. For each metric that changed, the delta from the value at
# the beginning of the period is logged. Also, the total values for
# all non-zero internal metrics are logged on shutdown. The default is true.
logging.metrics.enabled: true

# The period after which to log the internal metrics. The default is 30s.
logging.metrics.period: 30s

# Logging to rotating files. Set logging.to_files to false to disable logging to
# files.
logging.to_files: true
logging.files:
  # Configure the path where the logs are written. The default is the logs directory
  # under the home path (the binary location).
  path: /var/log/filebeat

  # The name of the files where the logs are written to.
  name: filebeat
```

Maintenant, ouvrez une nouvelle fenêtre de terminal et exécutez le script suivant pour simuler la charge sur votre serveur NGINX.

```
cd elastic-platform/app/
./artificial_load.sh
```

Lancer Kibana pour commencer à explorer les tableaux de bord qui ont été inclus dans le processus de configuration.

Accédez à Dashboard via le menu principal


![image.png](/elastic-tutorial/images/attachments/debutant/dashboard-menu.png)

Recherchez nginx et ouvrez le tableau de bord ECS Filebeat Nginx Overview.

![image.png](/elastic-tutorial/images/attachments/debutant/filebeat-dashboards.png)

Vous verrez quelque chose de similaire à la page ci-dessous.

![image.png](/elastic-tutorial/images/attachments/debutant/filebeat-nginx-dashboard.png)

Si vous rencontrez ce message dans une ou plusieurs visualisation.

![image.png](/elastic-tutorial/images/attachments/debutant/visualisation_error.png)

```
Saved field "user_agent.name" of data view "filebeat-*" is invalid for use with the "Terms" aggregation. Please select a new field.
```

Vous devez éditer la visualisation en cliquant sur le bouton "Editer" puis sélectionner la petite roue sur la visualisation.

Ensuite vous pouvez modifier la visualisation en écrivant le nom du champs et de sélectionner le champs avec .keyword.

sans keyword :

![image.png](/elastic-tutorial/images/attachments/debutant/edition_visualisation.png)

avec keyword :

![image.png](/elastic-tutorial/images/attachments/debutant/visualisation_keyword.png)

Vous pouvez enregistrer votre modification et vous aurez une visualisation qui fonctionne :

![image.png](/elastic-tutorial/images/attachments/debutant/visualisation_ok.png)

Faites défiler vers le bas et vérifiez quel est le navigateur le plus utilisé. Vous remarquerez que ce devrait être curl car il est utilisé par le script de simulation de charge.

![image.png](/elastic-tutorial/images/attachments/debutant/filebeat-nginx-most-used-browser.png)


Maintenant, cliquez sur Nginx access and error logs en haut du tableau de bord actuel pour voir le tableau de bord avec les journaux d'accès et d'erreurs que Filebeat a collecté de NGINX.

![image.png](/elastic-tutorial/images/attachments/debutant/filebeat-nginx-dashboard2.png)

Résumé : Dans ce laboratoire, vous avez exploré la facilité avec laquelle vous pouvez lire les fichiers journaux NGINX avec Filebeat et les indexer dans Elasticsearch. Vous avez également exploré les tableaux de bord Kibana et vu comment vous pouvez facilement surveiller les journaux NGINX.
