# Installation & Setup in GKE
I've installed my environment in GKE where you can get a 300 USD free credit to create Kubernetes Clusters. (https://console.cloud.google.com/freetrial). I do personally prefer GKE than minikube, this is a real Kubernetes cluster. 

In this section we will perform the next steps:
1) Create a Cluster
2) Install Istio in the istio-system namespace
3) Install KNative (Service & Eventing)
4) Create an Istio gateway to route inbound traffic (from outside the cluster to our services)

#### Creating the Cluster

I've created a 3 nodes cluster with a **n1-standard-2**  (2CPUs - 7.5GB RAM)setup for each node. I've selected Kubernetes version **1.12.6**  for the following installation. (Notice that with an older version, the one for default doesn't work)

You create the cluster and the connect to it (by going to http://console.cloud.google.com -> Kubernetes Engine -> Create a new Cluster and then Connect button on the cluster).

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

Once that's done let's install KNative Eventing (this will create two namespaces: knative-sources and knative-eventing)
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
cd extending-k8s-with-spring-cloud/
kubectl apply -f istio-gateway.yaml
```

