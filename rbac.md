# Setting up RBAC for our Controller
Because we are creating an Controller/Operator that is going to access the Kubernetes APIs from inside the cluster (as a Pod) we need to create 3 important resources: Role, RoleBinding and ServiceAccount. 

Then inside the k8s-operator/kubernetes/ directory:

```
kubectl apply -f cluster-role.yaml && \
kubectl apply -f cluster-role-binding.yaml && \
kubectl apply -f service-account.yaml

```

Once we have these resources configured, we can deploy our Operator, you can check that inside the deployment descriptor this Deployment is using the ServiceAccount that we have created before. 

> **Note**: notice that I've created a Cluster wide Role and Role Binding, both extremely permissive. You might want to check the official documentation to configure these resources to be as restrictive as possible for your Operator to work. [ServiceAccounts](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/), [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) 
