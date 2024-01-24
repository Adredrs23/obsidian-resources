
##### Is EKS load balancing different from external ALB/NLB load balancing?
Yes, the load balancing in Amazon Elastic Kubernetes Service (EKS) can involve both internal load balancing within the Kubernetes cluster and external load balancing for routing traffic from outside the cluster. Let's explore the key aspects of load balancing in EKS:

### 1. **Internal Load Balancing within the Cluster:**

#### a. Service Load Balancing:
   - **Kubernetes Service LoadBalancer:**
     - Within the Kubernetes cluster, EKS leverages Kubernetes services of type `LoadBalancer` to handle internal load balancing.
     - When you define a Kubernetes service of type `LoadBalancer`, EKS automatically provisions an internal Network Load Balancer (NLB) that distributes traffic to the pods associated with the service.

   - **Use Cases:**
     - Internal service-to-service communication within the EKS cluster.
     - Load balancing traffic across multiple pods of a service.

   - **Example Kubernetes Service:**
     ```yaml
     apiVersion: v1
     kind: Service
     metadata:
       name: my-service
     spec:
       selector:
         app: my-app
       ports:
       - protocol: TCP
         port: 80
         targetPort: 8080
       type: LoadBalancer
     ```

#### b. Ingress Controllers:
   - **Ingress Controllers:**
     - EKS supports Ingress controllers, such as the AWS ALB Ingress Controller, for external load balancing.
     - Ingress controllers provide additional features like path-based routing, SSL termination, and host-based routing.

   - **Use Cases:**
     - Routing external HTTP/HTTPS traffic to different services within the cluster.
     - Configuring advanced routing rules, SSL termination, etc.

   - **Example Ingress Resource:**
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: my-ingress
     spec:
       rules:
       - host: myapp.example.com
         http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: my-service
                 port:
                   number: 80
     ```

### 2. **External Load Balancing for Ingress Traffic:**

#### a. Application Load Balancer (ALB) Integration:
   - **ALB Integration:**
     - When using Ingress controllers, such as the AWS ALB Ingress Controller, EKS can automatically configure an external Application Load Balancer (ALB) to route external HTTP/HTTPS traffic to the services inside the cluster.

   - **Use Cases:**
     - Routing external HTTP/HTTPS traffic to different services based on paths, hosts, etc.
     - SSL termination, path-based routing, and other advanced features.

   - **Example ALB Ingress Controller Configuration:**
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: my-ingress
       annotations:
         kubernetes.io/ingress.class: alb
     spec:
       rules:
       - host: myapp.example.com
         http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: my-service
                 port:
                   number: 80
     ```

#### b. Network Load Balancer (NLB) Integration:
   - **NLB Integration:**
     - For specific use cases, EKS can also use Network Load Balancer (NLB) for external load balancing. This might be suitable for TCP/UDP-based traffic or scenarios where NLB features are preferred.

   - **Use Cases:**
     - Handling non-HTTP/HTTPS traffic, such as TCP or UDP.
     - High-performance and low-latency requirements.

   - **Example NLB Ingress Controller Configuration:**
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: my-ingress
       annotations:
         kubernetes.io/ingress.class: "nlb"
     spec:
       rules:
       - host: myapp.example.com
         http:
           paths:
           - path: /
             pathType: Prefix
             backend:
               service:
                 name: my-service
                 port:
                   number: 80
     ```

In summary, EKS provides mechanisms for both internal load balancing within the cluster (using Kubernetes services and Ingress controllers) and external load balancing for routing traffic from outside the cluster (using ALB or NLB). The choice between ALB and NLB depends on the specific requirements of your application, such as the type of traffic and desired features.