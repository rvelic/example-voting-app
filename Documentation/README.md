Challange: Leverage the advantage of containers
=========

This example is based on [Example Voting App](https://github.com/dockersamples/example-voting-app) and extends it with [ELK stack](https://www.elastic.co/what-is/elk-stack) for logging and monitoring as shown in `docker-compose.yml` where "vote" service logs to ELK for demonstration purposes. To run this, execute following in parent directory:

```
docker-compose up
```

To try the app you can:

1. Vote directly on [http://localhost:5000](http://localhost:5000)
2. View results on [http://localhost:5001](http://localhost:5001)
3. Logging and Metrics in Kibana on [http://localhost:5601](http://localhost:5601)

HA for the application can be observed using Docker Swarm (or Kubernetes) where 2 replicas of "vote" app are deployed. To run this locally, execute following in parent directory:


```
docker swarm init
docker stack deploy --compose-file docker-stack.yml vote
```

Notes
----

This solution is based on existing repository rather than "starting from scratch". I strongly believe deeply understanding and extending well documented and tested code is much better option if the objective is to deliver quality solution in short time.

Additionally, logging could be further extended by adding shipper for metric e.g. [Metricbeat](https://www.elastic.co/products/beats/metricbeat) (autodiscovery is available) and shipper for uptime monitoring e.g. [Heartbeat](https://www.elastic.co/products/beats/heartbeat). Both shippers are part of Elastic Beats products and so work well with our ELK stack. However, this could also be implemented using other services e.g. Grafana, Datadog, or perhaps as a SAAS service provided by a cloud provider.

The proposed solution includes `Jenkinsfile` that can be found in parent directory and defines build and push stage for each of our services. Those will rebuild changes in docker services and push them to our docker repository. Next step would be adding a deployment stage where we deploy our app to e.g. a Kubernetes service.

Assuming we're deploying to Azure Kubernetes Service from Azure Container Registry and we have deployment/service YAML files stored, kubectl configured on the Jenkins server, this additional stage could look something like this:

```
stage('Delpoying vote on Azure Kubernetes Service') {
    app = docker.image('testacr.azurecr.io/vote:latest')
    withDockerRegistry([credentialsId: 'acr_credentials', url: 'https://testacr.azurecr.io']) {
    app.pull()
    sh "kubectl create -f ."
    }
}
```