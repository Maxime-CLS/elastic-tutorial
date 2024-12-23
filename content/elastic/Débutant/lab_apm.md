---
title: "Lab - APM"
date: 2020-06-26T15:17:20+02:00
draft: false
weight: 15
---

Objectif : Dans ce laboratoire, vous apprendrez comment il est facile d'utiliser APM pour instrumenter les applications afin de collecter des informations détaillées sur les performances ainsi que les erreurs et les envoyer à votre déploiement Elasticsearch. Vous explorerez également l'application APM Kibana et verrez avec quelle facilité vous pouvez surveiller les performances des applications.

![image.png](/elastic-tutorial/images/attachments/debutant/apm/apm-architecture.png)

L'application Petclinic contient des variables d'environnement pour contacter le serveur APM :


```
  petclinic:
    image: docker.io/michaelhyatt/elastic-k8s-o11y-workshop-petclinic:1.25.0
    container_name: petclinic
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
```

Ces variables d'environnent font des références aux agents APM installés dans le code de l'application.

Maintenant que les trois microservices sont en cours d'exécution, accédez à la page Web de Petclinic et vous devriez voir la page d'accueil suivante :

![image.png](/elastic-tutorial/images/attachments/debutant/petclinic-home-page.png)

Cliquez sur FIND OWNERS et VETERINARIANS pour générer des données de performance à envoyer au serveur APM.

Cliquez sur ERROR pour générer des erreurs qui seront envoyées au serveur APM. 

**Vous avez constaté des dysfonctionnements ? Ne vous inquiétez pas, nous verrons cela par la suite.**

Lancez l'application APM pour commencer à explorer les données collectées.

Après avoir lancé l'application APM, vous accédez à l'aperçu des services.

![image.png](/elastic-tutorial/images/attachments/debutant/apm/apm.png)

Cliquez sur Détails du service pour explorer la santé globale du front-end en vérifiant les métriques sur les transactions, les erreurs et l'infrastructure.

![image.png](/elastic-tutorial/images/attachments/debutant/apm/petclinic-overview.png)

Ensuite, cliquez sur l'aperçu des transactions et sélectionnez le type de transaction http-request pour explorer les requêtes du frontend.

![image.png](/elastic-tutorial/images/attachments/debutant/apm/petclinic-react-transactions.png)

Ensuite, cliquez sur GET /api/vets pour explorer le traçage distribué et voir comment les applications et les services interagissent entre eux.

![image.png](/elastic-tutorial/images/attachments/debutant/apm/transaction-sample-id.png)

Ensuite, cliquez sur l'aperçu des erreurs pour explorer les erreurs de l'application.

![image.png](/elastic-tutorial/images/attachments/debutant/apm/petclinic-react-errors.png)

Ensuite, cliquez sur l'erreur "Aucun message disponible" pour voir la trace de la pile qui indique l'origine de l'erreur.

![image.png](/elastic-tutorial/images/attachments/debutant/apm/stacktrace.png)

Enfin, cliquez sur la transaction GET /api/error pour voir les erreurs associées et comment elles se sont propagées parmi les services.

![image.png](/elastic-tutorial/images/attachments/debutant/apm/transaction-sample-id.png)

Résumé : Dans ce laboratoire, vous avez appris comment il est facile de configurer des agents APM pour collecter des informations de trace et des erreurs afin de les indexer dans Elasticsearch. Vous avez également exploré l'application APM Kibana et vu avec quelle facilité vous pouvez surveiller les performances des applications.

