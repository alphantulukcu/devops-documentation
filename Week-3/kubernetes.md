# Deployments in Kubernetes & Helm

## Kubernetes

Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services. It facilitates declarative configuration and automation, supporting scenarios like microservices, CI/CD, big data processing, and machine learning.

## Kubernetes Architecture

![Kubernetes](../screenshots/kubernetes.png)


**Cluster Structure**:
A Kubernetes cluster consists of multiple nodes, with a master node managing the cluster's control plane and worker nodes running the containerized applications.

**Master Node Components**:
> **API Server**: The central management point for communication between cluster components, handling CRUD operations.

> **Scheduler**: Assigns newly created pods to appropriate worker nodes based on resource availability.

> **Controller Manager**: Runs control loops to ensure resources in the cluster are in the desired state.

> **etcd**: A distributed key-value store holding the state and configuration of the Kubernetes cluster.

**Worker Node Components**:

> **Kubelet**: Manages pods and containers on the node, ensuring they remain in the desired state.

> **Kube-Proxy**: Manages network traffic between pods and between pods and the external world.

> **Container Runtime**: Software that runs containers, such as Docker or containerd.

## **Kubernetes Objects**
> **Pods**: The smallest and basic unit in Kubernetes, representing a group of containers sharing the same network and storage resources.

> **Deployments**: Manage multiple copies (replicas) of an application, ensuring high availability and version control.

> **ReplicaSets**: Ensure a specified number of pod replicas are running at any given time.

> **Services**: Provide a stable IP address and DNS name to access a group of pods, enabling load balancing.

> **Persistent Volumes (PV) and Persistent Volume Claims (PVC)**: Handle storage resources in the cluster.

> **Secrets and ConfigMaps**: Manage sensitive data and application configurations, respectively.

> **Horizontal Pod Autoscaler (HPA)**: Automatically scales the number of pods based on resource utilization metrics.

> **Ingress**: Manages external access to services within the cluster, often routing HTTP/HTTPS traffic.

4. **Kubernetes Deployments**
   - **YAML Files & Kubectl**:
     - Deployments are typically managed using YAML configuration files, which are applied to the cluster using the `kubectl` command-line tool.

## Helm

Helm is a package manager for Kubernetes, used for defining, installing, and managing Kubernetes applications. Helm standardizes Kubernetes manifest files into charts, which can be versioned and reused.

**Installation**: Helm can be installed locally to manage Kubernetes applications.

**Common Commands**:

`helm create`: Creates a new Helm chart.

`helm install`: Installs a chart in the Kubernetes cluster.

`helm upgrade`: Upgrades an existing release to a new version.

`helm rollback`: Rolls back a release to a previous version.

`helm uninstall`: Removes a Helm release from the cluster.

### Helm Charts

**Chart.yaml**: Contains metadata about the chart, such as name, version, and description.

**Values.yaml**: Defines the default configuration values for the chart, which can be customized during deployment.

**Deployments with Helm**

Helm streamlines the deployment of applications to Kubernetes by bundling configurations into a single package, allowing easy installation and management of complex applications.

# Case

```deployment.yml````

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-case-1
  namespace: alphan
  labels:
    app: kubernetes-case-1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: kubernetes-case-1
  template:
    metadata:
      labels:
        app: kubernetes-case-1
    spec:
      containers:
      - name: kubernetes-case-1
        image: k8s-master-codecamp24.obss.io:30003/codecamp/alphantulukcu/kubernetes-case-1:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 5000
        livenessProbe:
          httpGet:
            path: /health
            port: 5000
          initialDelaySeconds: 3
          periodSeconds: 10
          timeoutSeconds: 5
      imagePullSecrets:
        - name: harbor-secret

```


```hpa.yml````

```yml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: kubernetes-case-1-hpa
  namespace: alphan
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: kubernetes-case-1
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70


```


```service.yml````

```yml
apiVersion: v1
kind: Service
metadata:
  name: kubernetes-case-1
  namespace: alphan
spec:
  selector:
    app: kubernetes-case-1
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30062
  type: NodePort

```