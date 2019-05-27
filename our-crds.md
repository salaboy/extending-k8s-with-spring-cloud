# Our CRDs
As shown in the previous diagrams, we will be defining 3 custom CRDs. Service A, Service B and Application (which aggregates these two types of services).

You can find our Custom Resource Definition in /extending-k8s-with-spring-cloud/crds/

We will first deploy the Custom Resource Definition for Service A

```
cd extending-k8s-with-spring-cloud/crds/ && \
kubectl apply -f service-a-crd-definition.yaml
```

This means that now we can do:
```
kubectl get a
```
Now we should get 
```
No resources found.
```
Due we haven't created any instance of this new resource type. 

We can now create a new ServiceA instance by creating a resource of type ServiceA:
```
kubectl apply -f my-a.yaml
```
Now we should be able to get the newly created ServiceA resource:

```
kubectl get a
```
Should return:
```
NAME   AGE
my-a   4s
```

You can now repeat for the CRD ServiceB:  
```
kubectl apply -f service-a-crd-definition.yaml 
```

Let's add the Application CRD now, which will aggregate ServiceA and ServiceB resources into an Application:
```
kubectl apply -f app-crd-definition.yaml
```

we should be able to test it with:

```
kubectl get apps
```

Once again, no resources found is ok.

Let's now the create  an Application resource:
```
k apply -f my-app.yaml
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

We can now create new resources from the following Kinds: ServiceA, ServiceB and Application, but since nobody is using these resources nothing will happen. We need now an Operator that understand about these CRDs and manage their lifecycle. 