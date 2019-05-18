# Extending Kubernetes with Spring Cloud
This repository serves as an index for a workshop like set of excersices about creating our custom resource definitions for Kubernetes using Spring Cloud components. 

During these excersices, you will learn about how to create your custom extensions, deploy them and use them as part of a Kubernetes Operator that will understand the domain specific restrictions that needs to be applied to the infrastructure. 

## Repositories
We will use the following repositories to build our services and our Kubernetes Operator:
- [K8s Operator](https://github.com/salaboy/k8s-operator)
- [Example Service A](https://github.com/salaboy/example-service-a)
- [Example Service B](https://github.com/salaboy/example-service-b)
- [Example Function A](https://github.com/salaboy/example-function-a)

It is recommended to clone all these repos under the same directory, as the following instructions are based in that assumption:
```
mkdir extending-k8s
cd extending-k8s
git clone https://github.com/salaboy/extending-kubernetes-with-spring-cloud
git clone https://github.com/salaboy/k8s-operator
git clone https://github.com/salaboy/example-service-a
git clone https://github.com/salaboy/example-service-b
git clone https://github.com/salaboy/example-function-a
```

## Infrastructure

For this tutorial we use Kubernetes, Istio and KNative to demonstrate what can be achieved with a Kubernetes Operator and how that might be done. An Kubernetes Operator doesn't require these technology stack, but due the adoption of these Kubernetes Extensions we believe that it is a good starting place.

### Installation

I've installed my environment in GKE where you can get a 300 USD free credit to create Kubernetes Clusters. (https://console.cloud.google.com/freetrial). I do personally prefer GKE than minikube, this is a real Kubernetes cluster. 

In this section we will perform the next steps:
1) Create a Cluster
2) Install Istio in the istio-system namespace
3) Install KNative (Service, Build, Eventing)

#### Creating the Cluster

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

Finally, instead of using Ingresses we will use an Istio Gateway to route traffic that is coming from outside the cluster to our services. 
For that in this repository we have the Gateway definition, so we can deploy it by running:

```
cd extending-kubernetes-with-spring-cloud/
kubectl apply -f istio-gateway.yaml
```


## Workshop
Steps Draft
- Deploy Service A
  - See how the service A returns the default answer if B is not present
  - Expose Service A with an Istio Virtual Service + Istio Gateway
- Deploy Service B
  - See how A start consuming B
  - Expose Service A with an Istio Virtual Service + Istio Gateway
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
You can clone [Example Service A](https://github.com/salaboy/example-service-a)

This project contains the source code for a very simple service that do the following:
- Every 5 seconds: call Service B
- Every 5 seconds: call Function A

It will print the output of each call or a message saying that the function or service were not available and 
it should return a default answer.

To deploy we need to:
Build the project with maven:
```
mvn clean install
```
Create a docker image with:
```
docker build -t salaboy/example-service-a:0.0.1 .
```

Then push the image to docker hub:
```
docker push salaboy/example-service-a:0.0.1
```
Finally deploy to our kubernetes cluster with:
```
cd kubernetes/
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Now in order to access our Example Service A service, we need to expose it to clients that are external to the Cluster. 
We can do this with Native Ingress resources or by using the Istio Gateway that is available to us. 

In order to expose our Service using Istio Virtual Services we should run:
```
kubectl apply -f istio-virtual-service.yaml

```
This will create a new Route to access service a at http <EXTERNAL IP>/service-a/ 


To deploy example service B follow the same instructions. 

### Creating and Deploying a Function with KNative Serving
You can clone [Example Function A](https://github.com/salaboy/example-function-a)

Build the project with 
```

mvn clean install
```
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


### Deploying our K8s Operator for Service A, B and Function A

Build the project with 
```
cd k8s-operator/
mvn clean install
```
Create a Docker image for it with
```
docker build -t salaboy/k8s-operator:0.0.1 .
```
Then push the docker image to be available in hub.docker.com
```
docker push salaboy/k8s-operator:0.0.1
```

Because we are creating an Operator that is going to access the Kubernetes APIs from inside the cluster (as a Pod) we need to create 3 important resources: Role, RoleBinding and ServiceAccount. 

Then inside the kubernetes/ 

``
kubectl apply -f cluster-role.yaml
kubectl apply -f cluster-role-binding.yaml 
kubeclt apply -f service-account.yaml
``

Once we have these resources configured, we can deploy our Operator, you can check that inside the deployment descriptor this Deployment is using the ServiceAccount that we have just created. 

**Note**: notice that I've created a Cluster wide Role and Role Binding, both extremely permissive. You might want to check the official documentation to configure these resources to be as restrictive as possible for your Operator to work. [ServiceAccounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/), [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) 


In order to deploy our K8s Operator we need to we can run:
```
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

We will first deploy the Custom Resource Definition

```
cd extending-kubernetes-with-spring-cloud/crds/
kubectl apply -f service-a-crd-definition.yaml
```

This means that now we can do:
```
kubectl get service-a
```
Now we should get 
```
No resources found.
```

Due we haven't created any instance of this new resource type. 

Let's add the Application CRD now:
```
kubectl apply -f app-crd-definition.yaml
```

we should be able to test it with:

```
kubectl get apps
```

Once again, no resources found is ok.

For the Operator to work, we need to create instance of these resources. We have a couple of instance definitions ready in the same directory, so let's do:

```
kubectl apply -f service-a.yaml
```
This create a new instance of our Custom Resource Definition ServiceA.

Now the following command should return a resource instance:
```
kubectl get a
```
Should return:
```
NAME        AGE
service-a   37s
```

Let's do the same with an Application resource:
```
k apply -f app.yaml
```

Now doing: 
```
kubectl get apps
```
should return:
```
NAME     AGE
my-app   47s
```


@TODO: when we deploy a new service A we can create a new virtual service to expose on the Istio Gateway.



# Conclusions

Explain the posibilities with KNative, Istio and Kubernetes native resources
