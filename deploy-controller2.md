
# Register watch on service

```
cd k8s-operator/

```

Let's switch to the **controller2** branch

```
git checkout controller2
```

Build the project with 

```
mvn clean install
```

Create a Docker image for it with

```
docker build -t salaboy/k8s-operator:controller2 .
```

Then push the docker image to be available in hub.docker.com.
> You will need a Docker Hub and replace **salaboy** with your user name. You might also need to do docker login before push.

```
docker push salaboy/k8s-operator:controller2
```

Finally deploy to our kubernetes cluster with:

```
cd kubernetes && \
kubectl apply -f deployment.yaml && \
kubectl apply -f service.yaml
```