# Extending Kubernetes with Spring Cloud
This repository serves as an index for a workshop like set of excersices about creating our custom resource definitions for Kubernetes using Spring Cloud components. 

During these excersices, you will learn about how to create your custom extensions, deploy them and use them as part of a Kubernetes Operator that will understand the domain specific restrictions that needs to be applied to the infrastructure. 

## Repositories
We will use the following repositories to build our services and our Kubernetes Operator:
- [K8s Operator](https://github.com/salaboy/k8s-operator)
- [Example Service A](https://github.com/salaboy/example-service-a)
- [Example Service B](https://github.com/salaboy/example-service-b)
- [Example Function A](https://github.com/salaboy/example-function-a)


## Infrastructure

For this tutorial we use Kubernetes, Istio and KNative to demonstrate what can be achieved with a Kubernetes Operator and how that might be done. An Kubernetes Operator doesn't require these technology stack, but due the adoption of these Kubernetes Extensions we believe that it is a good starting place.

### Installation

I've installed my environment in GKE where you can get a 300 USD free credit to create Kubernetes Clusters. (https://console.cloud.google.com/freetrial). I do personally prefer GKE than minikube, this is a real Kubernetes cluster. 

In this section we will perform the next steps:
1) Create a Cluster
2) Install Istio in the istio-system namespace
3) Install KNative (Service, Build, Eventing)

I've created a 3 nodes cluster with a **n1-standard-2**  (2CPUs - 7.5GB RAM)setup for each node. I've selected Kubernetes version **1.12.6**  for the following installation. (Notice that with an older version, the one for default doesn't work)

You create the cluster and the connect to it.

Then to install Istio (in the istio-system namespace):
```
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.4.0/istio-crds.yaml && \
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.4.0/istio.yaml
```

Wait for all the Istio component to start before installing KNative with
```
kubectl -n istio-system get pods -w
```
CTRL-C to terminate the watch 

Look for all the pods to be **Running**. 

When that is done you can install Knative Service (in the knative-service namespace)

```
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.4.0/serving.yaml
```
Once again, let's wait for all the services to start before proceeding.
Watch to all the 
```
kubectl -n knative-serving get pods -w
```

Once that is done, let's install KNative Build (in the knative-build namespace)
```
kubectl apply --filename https://github.com/knative/build/releases/download/v0.4.0/build.yaml
```

Watch with:

```
kubectl -n knative-build get pods -w
```

Finally let's install KNative Eventing (this will create two namespaces: knative-sources and knative-eventing)
```
kubectl apply --filename https://github.com/knative/eventing/releases/download/v0.4.0/release.yaml && \
kubectl apply --filename https://github.com/knative/eventing-sources/releases/download/v0.4.0/release.yaml && \
kubectl apply --filename https://raw.githubusercontent.com/knative/serving/v0.4.0/third_party/config/build/clusterrole.yaml
```

You can watch both:
```
kubectl -n knative-sources get pods -w
```
and 

```
kubectl -n knative-eventing get pods -w
```

With the infrastructure ready to go, we now can create our Kubernetes Deployments, Create our Functions and our Routing Policies with Istio.




## Workshop
Steps Draft
- Deploy Service A
  - See how the service A returns the default answer if B is not present
- Deploy Service B
  - See how A start consuming B
- Deploy Function A
  - See how Service A consume Function A
- Deploy Gateway (Explain why you might want to do that)
  - Need a tag in the k8s-operator called gateway just using discovery
  - Show basic Routing on K8s service discovery
  - You can hide and expose services based on business requirements, not yamls
  - Explain K8s security: ServiceAccount, Role & RoleBinding
- Show CRDs, Explain Application Concept
  - Deploy CRDs
  - Use kubectl to get the resources
- Deploy version 2 of k8s-operator
  - show code that watch resources changes 
  - Show custom routes creator

### Deploying a Service A


### Creating and Deploying a Function with KNative Serving
You can clone - [Example Function A](https://github.com/salaboy/example-function-a)

Build the project with mvn clean install
Create a Docker image for it with
```
docker build -t salaboy/example-function-a:0.0.1 .
```
Then push the docker image to be available in hub.docker.com
```
docker push salaboy/example-function-a:0.0.1
```
Then inside the kubernetes/ directory you can deploy your Knative function to your cluster with
``
kubectl apply -f service.yaml
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

# Conclusions

Explain the posibilities with KNative, Istio and Kubernetes native resources
