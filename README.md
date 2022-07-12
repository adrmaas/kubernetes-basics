# Kubernetes Basics
A brief introduction to kuberenetes

This intended as a quick introduction to Kubernetes.

# What’s Kubernetes?

It is an Orchestration platform: “A platform for automating deployment, scaling and operations of applications across clusters of hosts”

In other words it starts services or jobs and ensures that there are enough instances running. 

Removes the necessity of planning services manually: creating startup scripts, opening ports, assigning naming/IP addresses, etc,

# Kubernetes Components

There are many components that can make up Kubernetes and they may very between distributions but they all run on nodes or computes instances (think computer or VM).

There are two main node types: workers and control plane.

The control plane dynamically schedules software to run on worker nodes.

Affinity and projected resource consumption are a few of the factors that can be taken into consideration during scheduling.

# Interacting with Kubernetes

How do I talk to it?

`kubectl` is a command line tool that allows you to interact with Kubernetes clusters.

kubectl connects to several control plane API.

It’s possible to access the REST API directly.

There are GoLang, Python, and other client libraries.

# Kubernetes Objects

Objects or resources are defined in YAML.

Objects have a few common properties: apiVersion, kind, metadata, and spec.

YAML files containing one or more objects understood by the control plane are called “manifests”.

A manifest can hold multiple objects if the objects are separated by three dashes: ---

# Creating Objects

You can create objects or resources by applying a manifest.

To apply a manifest to a kubernetes cluster we would use kubectl apply subcommand to communicate with the Kubernetes control plane/API:

```
kubectl apply -f hello-world.yaml
```

# Types of Objects

Some common objects/resources include:
- Pods
- Services
- ConfigMaps
- Volumes
- Namespaces
- Ingress

# Pods

A Pod is the fundamental unit of deployment in Kubernetes. 

It is a container or group of containers which run in a shared context with shared storage, and shared network resources. 

Containers within a Pod can share storage and access the same volumes.  They also share an IP address and port space and can refer to each other using localhost.   Standard inter-process communications like System V semaphores or POSIX shared memory are available to containers in a Pod.

A simple pod manifest might appear as follows:
```
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
spec:
  containers:
  - name: hello-world
    image: hello-world:latest
    ports:
    - containerPort: 8080
```
# Just Pods?

So all I need is some pods?  Well no, not really.

Much in the same way that docker compose files are a better way of deploying containers than individual docker commands, in Kubernetes it is usual preferable to deploy pods via other objects such as Deployments, StatefulSets, DaemonSets, or Jobs which offer more management capability compared to directly deploying individual Pods.

# Services

A Service is a way of exposing a set of pods running an application as a load balanced service.  This provides a simple discover mechanism: you start a group of pods and they are reached via the hostname associated with the Service.

For example, we could have a backend  service made up of two or more backend pods.  The frontend could then reach these pods by contacting the service name (http://backend/some-uri/) and the connections would be round robin load balanced between the pods instances.

A simple service manifest might appear as follows:
```
apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  selector:
    app: hello-world
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
# Deployments

Deployments allow for the creation of sets of Pods and ReplicaSets and allows for declarative updates to those resources.  Updating a deployment will cause the Kubernetes Deployment Controller to change the actual state to the desired state at a controlled rate terminating old Pods and starting new Pods.  This allows for rollbacks as well as updates and provides a more controlled manner to deploy sets of Pods.

A simple service deployment might appear as follows:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world-deployment
  labels:
    app: hello-world
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: hello-world:latest
        ports:
        - containerPort: 8080
```
# ConfigMaps

ConfigMaps contain key value pairs of data that can be accessed by other objects in variety of ways.

ConfigMaps support multiline values.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: hello-world
data:
  # property-like keys; each key maps to a simple value
  hello_quantities: "3"
  world_name: "earth"

  # file-like keys
  hello-world.properties: |
    hello.quantities="3"
    world.name=5    
```
# VolumeMounts

A volumeMount is pretty much what it sounds like: it provides information on how a volume is mounted in the container and is part of the container object.

# LoadBalancer

A LoadBalancer is usually external the kubernetes cluster and part of the cloud provider’s capability.  The LoadBalancer exposes an Ingress or a Service to the outside world. 

# Ingress

While a LoadBalancer is usually a  Layer 4 IP load balancer from the cloud provider, an Ingress is a layer 7 component that handles fanning out of the entry into multiple service endpoints.

A simple ingress manifest might appear as follows:
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: hello-world
  rules:
  - http:
      paths:
      - path: /hello-world
        pathType: Prefix
        backend:
          service:
            name: hello-world
            port:
              number: 80
```
# Namespace

A Namespace object is a virtual cluster.  It limits the DNS visibility, naming separation, allows for standard naming of resources, and allows for the setting of resource limitations/quotas per Namespace.  A good use of a namespace would be dividing UAT and Production resources or dividing support third party applications from the core apps.

But the most important thing about a namespace is that deleting it will delete everything in the namespace so be careful!!!


# Controllers

A Controller is essentially a non-terminating control loop that maintains the desired state of the system.

Controllers track objects and respond to their specs to take action and change state closer to the desired state.  

Kubernetes has built in controllers but you can create your own.  

Controllers can run as a set of pods.  

A commonly deployed controller is cert-manager.

# Operators

Operators are follow the same Kubernetes control loop pattern as Controllers.

Often they use Custom Resource Definitions (CRDs).

These usually tie back to pods that are clients of the Kubernetes API.

When certain resources/objects are created they perform tasks to bring the cluster to a desired state but focus on a single app.

Operators are often used to setup complex configurations.

# Handy Commands

Some common commands.

Get a list of contexts that you have configured:
```
kubectl config get-contexts
```
Get a list of namespaces from a context/cluster:
```
kubectl --context cluster-name get namespaces
```
Get pods in a namespace from a cluster:
```
kubectl --context cluster-name --namespace name get pods 
```
Describe a pod to get information about it:
```
kubectl --context cluster-name --namespace name describe pod hello-world-8rlgb
```
Get the logs from a pod:
```
kubectl --context cluster-name --namespace name logs -f hello-world-8rlgb
```
Execute:
```
kubectl --context cluster-name --namespace name exec --stdin --tty hello-world-8rlgb -- /bin/bash
```
Apply a manifest:
```
kubectl --context cluster-name --namespace name apply -f manifest.yaml 
```
Delete resources in a manifest:
```
kubectl --context cluster-name --namespace name delete -f deploy.yaml
```
Graphic Interface

Often I try and persuade people of value of the command line but in Kubernetes what you don’t see can hurt you so I fully recommend using the command line and a GUI.

[Lens](https://k8slens.dev/) is one such GUI but requires creating an account with their service.

[KubeNav](https://github.com/kubenav/kubenav) is a more recent entry that 9at the time of writing) required giving away less information than Lens.
