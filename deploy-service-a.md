# Deploying Service A

This project contains the source code for a very simple service that do the following:
- Every 10 seconds: call Service B (initial delay 5 seconds)
- Every 10 seconds: call Function A

It will print the output of each call or an error message if the services are unavailable.

Let's build and deploy our Service A
```
cd example-service-a/
```


To deploy we need to:
Build the project with maven:
```
mvn clean install
```
Create a docker image with:
```
docker build -t salaboy/example-service-a:0.0.1 .
```


Then push the docker image to be available in hub.docker.com.
> You will need a Docker Hub and replace **salaboy** with your user name. You might also need to do docker login before push.

```
docker push salaboy/example-service-a:0.0.1
```
Finally deploy to our kubernetes cluster with:
```
cd kubernetes/ && \
kubectl apply -f deployment.yaml && \
kubectl apply -f service.yaml
```
You can take a look at the logs by doing:

```
kubectl logs -f my-service-a-<pod-hash> my-service-a
```

You can find the pod name by ruunning ```kubectl get pods```

## Exposing Service A
Now in order to access our Example Service A service, we need to expose it to clients that are external to the Cluster. 
We can do this with Native Ingress resources or by using the Istio Gateway that is available to us. 

In order to expose our Service using Istio Virtual Services we should run:
```
kubectl apply -f istio-virtual-service.yaml

```
This will create a new Route to access service a at http <EXTERNAL IP>/my-service-a/ 