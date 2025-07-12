# reviewing-kubernetes-concepts

### Kubernetes Nodes
Nodes are simply, worker machines where kubernetes launch the containers. A group of kubernetes nodes are called cluster.
**Master Node, or simply, Control Plane** watches the nodes and manages the workload of worker nodes.

### Kubernetes components
Installing Kubernetes on a system actually installs the below components on a system.
#### API Server
Acts as the frontend of Kubernetes. Provides the APIs for all kind of UI tools to talk to the kubernetes cluster. It resides usually in the master node.
#### etcd Keystore
Distributed reliable key-value store to keep all the data within kubernetes cluster.
#### Scheduler
Responsible for distributing workloads (containers) among multiple nodes. It looks for newly created containers and assign them to nodes.
#### Controller
The brain behind the orchestration. If any node / container / endpoint goes down, it takes the decision to bring up the new container or the node.
#### Container Runtime
Where containers run. For example, docker, rocket, cri-o, podman etc.
#### Kubectl
An agent that runs on each node of the cluster. It makes sure that the container is running as expected. 
It usually resides in worker nodes and communicate with kube-apiserver in the master node.

### Kubernetes component workflow
![Kubernetes components workflow.png](images/kubernetes-components-workflow.png)

### Common Kubernetes Objects

Every kubernetes object has 4 root level fields:

- **apiVersion:** Version of the **kind** definition
- **kind:** Can be Pod, Deployment, ReplicaSet, ConfigMap, Service, Ingress, Job, etc.
- **metadata:** Contains metadata of a manifest denition, can be used by other objects.
- **spec:** Specification based on which the object will be created
#### Pod
Consists of at least 1 container. Sometimes, can have sidecar (helper) containers as well.

**Example manifest definition:**
```nging-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```
To run the pod:

```shell
kubectl apply -f nginx-pod.yaml
```

To see all the running pods, you can run:
```shell
kubectl get pods
```
To inspect the pod definition:
```shell
kubectl describe pod nginx-pod
```
#### Deployment
Contains one or more pod definition.

**Example manifest definition**
```deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80

```
### Basics of Kubernetes manifest files
Kubernets manifest files contain 