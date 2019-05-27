
# Deploying Function A

```
cd example-function-a/
```

Build the project with 
```
mvn clean install
```

Create a Docker image for it with

```
docker build -t salaboy/example-function-a:0.0.1 .
```

Then push the docker image to be available in hub.docker.com.
> You will need a Docker Hub and replace **salaboy** with your user name. You might also need to do docker login before push.

```
docker push salaboy/example-function-a:0.0.1
```
Then inside the kubernetes/ directory you can deploy your Knative function to your cluster with
``
kubectl apply -f kservice.yaml
``

You can find the external IP that you can use to call your function:
```
kubectl get svc istio-ingressgateway -n istio-system
```

And then call the function: 

```
http <EXTERNAL IP> 'Host:example-function-a.default.example.com'
```

You should see the function returning a value. 


> Notice that you can customize how the autoscaler scale down your function pods with: 
> ```
> kubectl edit cm config-autoscaler -n knative-serving
>  ```
> and changing the default value to:
> ```
> scale-to-zero-grace-period: "30s"
> ```
