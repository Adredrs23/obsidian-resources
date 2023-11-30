Docker se containers to bana liye usko manage kon karega ab???
Kubernetes help you monitor your containers and manage them with very less overhead.

[Kubernetes](https://kubernetes.io/docs/concepts/overview/), also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications.

Some key points:
- Hassle free Rollouts and rollbacks
- Service Discovery and Load balancing
- Self healing
- Storage Orchestration
- Secret & configuration management
- Scaling & speedy deployment of complex apps
- Infra configuration

### K8s Cluster

Worker Nodes 
- At least one WN
- used to run the containers

Pods
- Every container is wrapped around a pod
- 1:1 Container : pod
- can have multiple containers but not preferred

Kubelet
- Manages the containers inside a worker node
- communicates with the master node

Kube Proxy
- maintains network rules,
- inbound and outbound node communication rules

Master Nodes
- one or many
- manages worker nodes and the containers running within them
- API Server - used to comm with Kubelet, GUI or KubeCTL - which will communicate with the Worker Nodes
- Scheduler - schedule pods, decide where to place the pod in which WN, Evict pods
- Controller manager - detect changes, notifies changes

![[Kubernetes Cluster Diagram.png]]

### Commands
Cheat sheet : https://www.bluematador.com/learn/kubectl-cheatsheet

```bash

kubectl get all
kubectl get pod
kubectl describe pod-name

kubectl delete resource-name pod-name
# for pods one can omit the resource-name
# deletion will spin up another pod - so 2 pods with terminating and running state as k8s configuration specify atleast one should be running

# imperative way of creating resources
kubectl create resource-name

# declarative way of creating resources (Preferred)
kubectl apply -f manifest-file.yaml

# to delete everything and I mean everything
kubectl delete all --all 
```

### Manifest files
- apiVersion: where are the resources exported from. Something like folders from a library eg @emotion/styled, @emotion/react.

```yml
# This will only run up a pod but it wont be accessible, we need a service for that

apiVersion: v1
kind: Pod
metadata:
	name: client-pod
	labels:
		app: client

spec:
	containers:
		- name: client
		  image: image-name:tag

```
### Services
- Nodeport (Development only) (exposing pods to outer systems)
- ClusterIp (Used for inter pods communication in a cluster)
- LoadBalancer (Prod) (exposing pods to outer systems, along with Load balancing)
- Ingress (Dev & Prod)

![[NodePort Service flow.png]]
![[NodePort Service Diagram.png]]
```yml

apiVersion: v1
kind: Service
metadata:
  name: client-srv
spec:
  type: NodePort
  selector:
    app: client
  ports:
    - port: 3000
      targetPort: 3000 # the one that container exposes
      nodePort: 30007 # where local machine will access it
```

```yaml

# Template snippet from Kubernetes docs
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - port: 80
      # By default and for convenience, the `targetPort` is set to
      # the same value as the `port` field.
      targetPort: 80
      # Optional field
      # By default and for convenience, the Kubernetes control plane
      # will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```
```
```


![[ClusterIp Diagram.png]]

- ClusterIPs have internal DNS managed under a different namespace so it is present but invisible
- Everytime a CIP service goes down, it comes up with a new IP
- Hence, DNS mapping is required.
- Use the service name to refer to the IPs and not the actual IP

```yml
# Example of ClusterIP

apiVersion: v1
kind: Service
metadata:
  name: mongo-srv
spec:
  type: ClusterIP # is the default service if not specified
  selector:
    app: mongo # for this pod label
  ports:
    - port: 27017
      targetPort: 27017

# connect using mongo://mongo-srv/collection-name
```

### Namespaces
```bash
kubectl get namespaces

kubectl get all -n namespace-name

# All actions are applied to default namespaces if not defined
```