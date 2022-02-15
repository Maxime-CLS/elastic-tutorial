---
title: "Installation de l'Agent Elastic"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 5
---

Modifier la configuration par défaut des agents elastic via le serveur fleet sur l'interface Kibana :

**Management** -> **Fleet**

Cliquez sur **Fleet settings**

![image.png](/images/attachments/debutant/fleet_settings.png)

**Fleet Server Hosts**

```
https://fleet-server:8220
```

**Elasticsearch Output Configuration (YAML)**

```
ssl.certificate_authorities: ["/usr/share/elasticsearch/config/certificates/ca/ca.crt"]
```

![image.png](/images/attachments/debutant/fleet_settings_configuration.png)


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


![image.png](/images/attachments/debutant/fleet_agent_elastic.png)


### Intégration du module Docker

Cliquez sur **Agent policies** on niveau de l'interface Kibana (Fleet).

![image.png](/images/attachments/debutant/agent_policies.png)

Cliquez sur **Default Policy**.

![image.png](/images/attachments/debutant/default_policy.png)

Cliquez sur **Add Integration**.

![image.png](/images/attachments/debutant/add_integration.png)

Recherchez dans la barre **docker**.

![image.png](/images/attachments/debutant/search_integration_docker.png)

Ajoutez le module Docker.

![image.png](/images/attachments/debutant/add_docker_integration.png)

Sauvegarder votre choix.

![image.png](/images/attachments/debutant/add_docker_integration_save.png)

Vous devez obtenir ce résultat dans l'interface Kibana.

![image.png](/images/attachments/debutant/new_default_policy.png)

