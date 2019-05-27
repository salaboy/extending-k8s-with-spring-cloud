# Deploying our K8s Operator

Let's switch to the **operator** branch
```
git checkout operator
```

Build the project with 
```
mvn clean install
```
Create a Docker image for it with
```
docker build -t salaboy/k8s-operator:operator .
```
Then push the docker image to be available in hub.docker.com.
> You will need a Docker Hub and replace **salaboy** with your user name. You might also need to do docker login before push.

```
docker push salaboy/k8s-operator:operator
```

In order to deploy our K8s Operator we need to we can run:
```
cd kubernetes && \
kubectl apply -f deployment.yaml
```

If you take a look at the logs of the K8s Operator you will see that the pod is running, but it lacks the right resources to operate:
```
kubectl logs -f k8s-operator-<POD name> k8s-operator

// you should see something like:
> Custom CRDs required to work not found please check your installation!
```

This means that we need to provide the Operator the Custom Resource Definitions we need to deploy these resources to make them available to our Cluster. In the crds/ directory we will find two things:
1) Custom Resource Definitions
2) Custom Resource (instance) 