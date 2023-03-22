Kubernetes guide =====

::: {#installation}
Creating a cluster with minikube on host machine:
:::

``` console
minikube start

kubectl version --output=yaml
kubectl cluster-info
```

View nodes in the cluster \-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

``` console
kubectl get nodes
```

# Deployment:

``` console
kubectl create deployment <name> --image<image-name-location>
e.g. => kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```

List your deployments: \-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

``` console
kubectl get deplyments
```

Pods that are running inside Kubernetes are running on a private,
isolated network. By default they are visible from other pods and
services within the same kubernetes cluster, but not outside that
network. When we use kubectl, we\'re interacting through an API endpoint
to communicate with or application. \-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\--

In another terminal, run:

``` console
echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response.
Please click the first Terminal Tab\n"; 
kubectl proxy
```

In another terminal, run:

``` console
curl https://localhost:8001/version
```

The API server will automatically create an endpoint for each pod, based
on the pod name, that is also accessible through the proxy. First we
need to get the Pod name, and we\'ll store in the environment variable
POD_NAME:

------------------------------------------------------------------------

``` console
export POD_NAME=$(kubectl get pods -o go-template --template
 '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
```

You can access the Pod through the API by running:

``` console
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/ 
```

------------------------------------------------------------------------

Pod examination and configuration =====

Pod status can be seen via the describe command of kubectl.

``` console
kubectl describe pod <pod-name>
```

Access to the contairer inside the Pod can be granted via the following
command, which iterrates an interactive terminal.

``` console
kubectl exec -it <pod-name> -- bin/bash
```

------------------------------------------------------------------------

Deployments ======

The Deployment commands are the following:

``` console
kubectl create deployment [name]
kubectl edit deployment [name]
kubectl delete deployment [name]
```

------------------------------------------------------------------------

Status of different K8s components

``` console
kubectl get nodes|pod|services|replicaset|deployment
```

------------------------------------------------------------------------

Debuggin pods

``` console
kubectl logs [pod name]
kubectl exec -it [pod-name] -- bin/bash
```

------------------------------------------------------------------------

Creating a custom configuration file

------------------------------------------------------------------------

Configuration files in K8s are of .yaml file format. After a Pod,
Container and Deployment are created, a config file can be
created/edited.

Creating a .yaml configuration file:

``` console
touch nginx-deployment.yaml
vim nginx-deployment.yaml
```

Inside the .yaml file, a strict and specific syntax must be followed.

``` console
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:       ##specification for the deployment
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:   ##specification for the pods
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

------------------------------------------------------------------------

===== User configuration files for CRUD After the config file has been
created, it can be applied via the following command:

``` console
kubectl apply -f nginx-deployment.yaml
```

If we run the following command, we can see the new deployment is ready
and running:

> kubectl get pod

Similarly for the deployment:

``` console
kubectl get deployment
```
