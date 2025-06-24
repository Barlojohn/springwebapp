# springwebapp
Spring Boot application used to demonstrate deployment to Kubernetes (minikube) along with Kafka and Zookeeper. 
# Overview
The application provides a REST endpoint that accepts a request to trigger sending events. The number of events to produce can be specified.

The application also consumes an event that triggers sending outbound events. The number of events to produce can be specified.

Kafka, Zookeeper, and the demo Spring Boot application are deployed as Docker containers to minikube, which is an implementation of Kubernetes that is great for local testing. Interacting with the application and Kafka are demonstrated by calling the REST endpoint to trigger emitting events, and by sending events to Kafka and receiving the emitted events from the application via the Kafka commandline tools.

# Walkthrough

### Run minikube

Start the cluster
```
minikube start
```

### Deploy the helm chart

```
helm install kafka_webapp-0.9.0.tgz
```

### After the system is up and running expose the spring boot app. This will open the root page of the Spring Boot application in the browser, which displays Spring Boot Demo

```
minikube service kafka-kubernetes-demo-service --namespace demo
```

### Running the Demo, interacting via REST

```
curl -X GET http://{EXPOSED_IP}:{PORT}/v1/demo/version
```

### In order to trigger the application into sending three outbound events to Kafka, send a REST request to the following URL(The response should be a 200 Success):

```
curl -v -d '{"numberOfEvents":3}' -H "Content-Type: application/json" http://{EXPOSED_IP}:{PORT}/v1/demo/trigger
```

### Check the logs from kafka-kubernetes-demo pod

```
kubectl logs kafka-kubernetes-demo-7795456b4d-9kp6f -n demo
```

## Interacting via Events

```
kubectl exec -it kafka-deployment-6cc5777dd5-jnj8k -n demo -- bash
```

### Kafka Consumer - Run at pod (You should see the 3 entries sent before from curl / REST)

```
kafka-console-consumer --bootstrap-server localhost:9092 --topic demo-outbound-topic --from-beginning
```

### Open a new terminal and execute : kubectl exec -it kafka-deployment-6cc5777dd5-jnj8k -n demo -- bash

```
kubectl exec -it kafka-deployment-6cc5777dd5-jnj8k -n demo -- bash
```

### Kafka Producer - Run at pod

```
kafka-console-producer --broker-list localhost:9092 --topic demo-inbound-topic
```

### Enter 2 events to kafka consumer

```
{"numberOfEvents":2}
```

### This must be reflected at Kafka consumer and Spring Boot application logs.

