Docker se containers to bana liye usko manage kon karega ab???
Kubernetes help you monitor your containers and manage them with very less overhead.

[Kubernetes](https://kubernetes.io/docs/concepts/overview/), also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications.

Some keypoints:
- Hassle free Rollouts and rollbacks
- Service Discovery and Load balancing
- Self healing
- Storage Orchestration
- Secret & configuration mnmgt
- Scaling & speedy deployment of complex apps
- Infra configuration

### K8s Cluster

Worker Nodes 
- Atleast one WN
- used to run the containers

Pods
- Every container is wrapped around a pod
- 1:1 Container:pod
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
