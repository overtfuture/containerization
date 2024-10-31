# Kubernetes

Welcome 👋🏻, this is an introductory guide to setting up and running Kubernetes locally for general testing, experimenting, or playing around with Kubernetes workflows.

## Overview

Kubernetes is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications across a cluster of machines. While Docker and Docker Compose are excellent for packaging applications into containers and running multi-container setups on a single host, Kubernetes extends these capabilities to a distributed system. It handles the complexities of coordinating multiple containers running on multiple hosts, providing features like automatic scaling, load balancing, self-healing, and seamless rollouts and rollbacks.

By transitioning to Kubernetes, you can manage applications with greater resilience and scalability. Kubernetes abstracts the underlying infrastructure, allowing for consistent deployments across various environments—be it on-premises, cloud, or hybrid setups. Its robust ecosystem and declarative configuration approach make it a powerful tool for running production-grade workloads, especially as your applications and teams grow in complexity.

## Installation

Getting started with Kubernetes is really easy. There are a couple really versatile projects that allow for running Kubernetes quickly and easily. More projects are available, but to keep things simple, these two are common and cross platform.

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
    - Simplest way to get started
    - Toggle on the Docker Desktop Kubernetes in settings
- [Minikube](https://minikube.sigs.k8s.io/docs/)
    - Very portable
    - Just requires Docker, but runs anywhere Docker can
    - More closely resembles cloud Kubernetes with built in tools and addons

## Setup

Both options have getting started guides:

- [Docker Desktop Kubernetes](https://docs.docker.com/desktop/kubernetes/)
- [Minikube Getting Started](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Fx86-64%2Fstable%2Fbinary+download#LoadBalancer)

## Applications

Deploying applications on a Kubernetes cluster can be done through various methods, each with its own strengths, depending on the complexity of the application and the needs of the development team. Here’s a list and summary of common ways to deploy Kubernetes applications:

### Kubernetes YAML Manifests

**Description**: The most basic and direct method involves creating YAML files that define Kubernetes resources such as Pods, Deployments, Services, ConfigMaps, Secrets, etc.

**Use Case**: Ideal for simple applications or when you need precise control over Kubernetes resources.

**Process**: Create YAML files for each resource, then use kubectl apply -f <file>.yaml to deploy them.

**Pros**: Full control over resource configuration; integrates directly with Kubernetes.

**Cons**: Can become complex and verbose as applications grow.

### Helm Charts

**Description**: Helm is a package manager for Kubernetes that allows you to define, install, and upgrade complex applications using Helm charts.

**Use Case**: Great for deploying complex applications with many interdependent components or for managing configuration variations between environments.

**Process**: Create a Helm chart, then use helm install <chart> to deploy. Helm charts can be stored in public or private repositories.

**Pros**: Simplifies deployment of complex applications; supports versioning and easy upgrades; templates can reduce YAML verbosity.

**Cons**: Learning curve for writing Helm templates; requires managing Helm releases.

### Kustomize

**Description**: A native Kubernetes configuration management tool that allows you to customize Kubernetes YAML files without needing templating.

**Use Case**: Useful for customizing base YAML files for different environments (e.g., dev, staging, production).

**Process**: Define a base YAML configuration, and use overlays to adjust the settings for different environments. Deploy using kubectl apply -k <directory>.

**Pros**: Built into kubectl; no templating language needed; good for managing variations across environments.

**Cons**: Less powerful than Helm for managing complex deployments; can be difficult to manage very large configurations.

### GitOps (e.g., Argo CD, Flux)

**Description**: GitOps uses Git as a single source of truth for managing Kubernetes deployments. Changes are made by pushing updates to a Git repository, and tools like Argo CD or Flux synchronize those changes to the Kubernetes cluster.

**Use Case**: Ideal for teams practicing CI/CD with version control; enables easy rollbacks and history tracking.

**Process**: Set up a Git repository for application configurations, and configure a GitOps tool to monitor and apply changes.

**Pros**: Strong version control and audit trail; integrates well with CI/CD pipelines.

**Cons**: Requires additional setup of GitOps tools; may add complexity to the deployment process.

## Storage

Step-by-Step Example: Using a PVC with Dynamic Provisioning

- Create a PersistentVolumeClaim (PVC):
  - The PVC will automatically request storage using the default StorageClass available in your cluster (e.g., in Minikube).
  - accessModes: ReadWriteOnce allows the volume to be mounted as read-write by a single node.
  - resources.requests.storage: Requests 500Mi of storage.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

- Use the PVC in a Pod:
	- Create a Pod that mounts the PVC, allowing the container to use the dynamically provisioned storage.
	- volumeMounts: Mounts the PVC at the specified path (/usr/share/nginx/html) inside the container.
	- volumes: Refers to the PVC (example-pvc), allowing the Pod to use the claimed storage.

- Deploy the Resources:
	- Apply the YAML files to create the PVC and Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
    - name: nginx
      image: nginx:1.21
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: storage-volume
  volumes:
    - name: storage-volume
      persistentVolumeClaim:
        claimName: example-pvc
```


```shell
kubectl apply -f pvc.yaml
kubectl apply -f pod.yaml
```

- These commands will create the PVC, dynamically provision the necessary PV through the default StorageClass, and then use the PVC to provide storage to the Pod.

- Verify the Setup:
  - Check the status of the PVC and the associated PV:

```yaml
kubectl get pvc
kubectl get pv
kubectl get pods
```

- `kubectl get pvc` should show that the PVC is Bound to a dynamically created PV.
- `kubectl get pv` should show the PV as Bound to the PVC.
- `kubectl get pods` should show the Pod running and using the storage.

## Ingress

To create a basic Ingress using the Kubernetes NGINX Ingress controller included with Minikube, follow these steps. This example assumes you have Minikube running with the NGINX Ingress controller enabled.

Prerequisites:

- Enable the NGINX Ingress Controller in Minikube:
If you haven’t enabled the NGINX Ingress controller in Minikube yet, run:

`minikube addons enable ingress`

- Verify that the Ingress controller is running:

`kubectl get pods -n ingress-nginx`

Step-by-Step Example: Basic Ingress Setup

Create a simple nginx deployment to serve a sample page

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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
          image: nginx:1.21
          ports:
            - containerPort: 80
```

Create a Service, expose the NGINX Deployment using a Service of type ClusterIP.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Create an Ingress Resource, define an Ingress resource that routes HTTP traffic to the NGINX Service.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
    - host: nginx-example.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```

This Ingress will route traffic to nginx-service for requests made to http://nginx-example.local.

Deploy the Resources:

Apply the YAML files to create the Deployment, Service, and Ingress:

```shell
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f ingress.yaml
```

Update `/etc/hosts` (Optional for Local Testing):

To test the Ingress locally on Minikube, add an entry to your /etc/hosts file (or C:\Windows\System32\drivers\etc\hosts on Windows) to point nginx-example.local to the Minikube IP:

`echo "$(minikube ip) nginx-example.local" | sudo tee -a /etc/hosts`

Replace `$(minikube ip)` with the IP address of your Minikube instance, which can be found using:

`minikube ip`


Test the Ingress:

After the /etc/hosts update, you should be able to access the NGINX application using a web browser or curl:

`curl http://nginx-example.local`

This should return the default NGINX welcome page.
