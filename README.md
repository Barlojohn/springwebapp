# springwebapp
Spring Boot application used to demonstrate deployment to Kubernetes (minikube) along with Kafka and Zookeeper. 
# Overview
The application provides a REST endpoint that accepts a request to trigger sending events. The number of events to produce can be specified.

The application also consumes an event that triggers sending outbound events. The number of events to produce can be specified.

Interacting with the application and Kafka are demonstrated by calling the REST endpoint to trigger emitting events, and by sending events to Kafka and receiving the emitted events from the application via the Kafka commandline tools.

# Walkthrough

### Run minikube

Start the cluster
```
minikube start
```
![image](https://github.com/user-attachments/assets/7c60c509-5afb-44f6-a2f9-f03c08677588)



### Deploy the helm chart

```
helm install springwebapp kafka_webapp-0.9.0.tgz
```
![image](https://github.com/user-attachments/assets/1ed5d49e-064e-4d9c-90c4-fab350bb8343)




### Some minutes later the pods should be up and running:
![image](https://github.com/user-attachments/assets/253624e5-ab4b-4e1a-9175-0d9103879433)




### After the system is up and running expose the spring boot app. This will open the root page of the Spring Boot application in the browser, which displays Spring Boot Demo

```
minikube service kafka-kubernetes-demo-service --namespace demo
```
![image](https://github.com/user-attachments/assets/545cf61b-ba39-43f6-bdd6-997080bf6ede)




### Running the Demo, interacting via REST

```
curl -X GET http://{EXPOSED_IP}:{PORT}/v1/demo/version
```

### In order to trigger the application into sending three outbound events to Kafka, send a REST request to the following URL(The response should be a 200 Success):

```
curl -v -d '{"numberOfEvents":3}' -H "Content-Type: application/json" http://{EXPOSED_IP}:{PORT}/v1/demo/trigger
```
![image](https://github.com/user-attachments/assets/f6a3a0ed-700a-4a28-86a4-5ecf715539cc)




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
![image](https://github.com/user-attachments/assets/dab10b10-6cf3-45f3-9ed3-a8543d74d24d)




### Open a new terminal and execute : 

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
![image](https://github.com/user-attachments/assets/7a770728-3bcf-4784-8634-76b1d0033979)




### This must be reflected at Kafka consumer and Spring Boot application logs.
![image](https://github.com/user-attachments/assets/4957ac5f-d2d9-47e4-b560-ee37737b5944)

![image](https://github.com/user-attachments/assets/6ec23337-3a65-4704-bc59-50971530f234)



