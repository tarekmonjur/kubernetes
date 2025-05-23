
# Kubernetes
### kubernetes known as K8s develop by Google at 2014 based on GoLangs

* High availability and No downtime
* Scalability and High performance
* Recovery, backup and Restore

<br/>

## Container
A technology that bundles the code for an application and the configuration required to runt he code itself in out unit.

* Portable.
* Use less CPU and memory then vairtual machine.
* Self contained, Can be spun up and down in a second.
* Quick replicatioin.

### Container Registry:
A database that store container images like

* Docker
* Podman
* Containered
* rkt
* LXD

<br/>

## Image
A container image is a file with executable code that can be run as a continer.

<br/>

## K8s Basic Components

* POD
* Service
* Ingress
* Namespace
* External Configuration
* Volumes / Storage
* Replica
* Deployment

<br/>

## POD

* **Smallest unit of kubernetes**
* **POD contain containers**
* **POD share storage, Network, resources and specification for running the containers**
* **Usually one POD use for one application**
* **Each POD gets its own IP address**
* **Always get new IP on re-creation**


<br/>

# Namespaces
Kubernetes namespace let us isolate and organize our workloads. when we use kubernetes the we probably want to create a different namespaces to organize our applications and microservices.
<br/>
For example if we have development and production environments running in the same kubernetes cluster, we can separate our applications running in each environment by deploying them into a namespace called development, and a different one called production.
<br/>

### Get Namespaces command
```
kubectl get namespaces
```

### Create namespace by Yaml config file
```
---
apiVersiion: v1
kind: Namespace
metadata:
    name: development
---
apiVersiion: v1
kind: Namespace
metadata:
    name: production
```

### Create namespace command
```
kubectl apply -f namespace.yaml
```

### Delete namespace command
```
kubectl delete -f namespace.yaml
```

# Deployment

### Create pod deployment by deployment.yaml file
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: 01-pod-info-deployment
  namespace: development
  labels:
    app: pod-info
spec:
  replicas: 2
  selector:
    matchLabels:
      app: pod-info
  template:
    metadata:
      labels:
        app: pod-info
    spec:
      containers:
      - name: 01-pod-info-container
        image: kimschles/pod-info-app:latest
        ports:
        - containerPort: 3000
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
```

### POD deployment command
```
kubectl apply -f deployment.yaml
```

### Get POD deployment command
```
kubectl get deployments -n development
```

### Get PODs command
```
kubectl get pods -n development
```

### Get PODs with extra info command
```
kubectl get pods -n development -o wide
```

### Delete POD command
```
kubectl delete pod <pod_name> -n development
```

### Describe POD description
```
kubectl describe pod <pod_name> -n <namespace_name>
```

### Inside into the POD command
```
kubectl exec -it <pod_name> -- /bin/sh
```

### Show the POD logs command
```
kubectl logs -f <pod_name>
```

### Create deployment for busybox
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
  namespace: default
  labels:
    app: busybox
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox-container
        image: busybox:latest
        # Keep the container running
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep 30; done;" ]
        resources:
          requests:
            cpu: 30m
            memory: 64Mi
          limits:
            cpu: 100m
            memory: 128Mi
```

***
## **Challenge:** Create your own development
* ***Create a new development in a file called quote.yaml***
* ***Name the deployment and name the app label quote-service***
* ***Use the development namespace***
* ***Name the container quote-container***
* ***Run two replicas***
* ***Use the image datawire/quote:0.5.0***
* ***Set the container to accept traffic at port 8080***
* ***Create the pods using kubectl apply -f quote.yaml***
* ***Use BusyBox to test that the application can accept traffic from inside the cluster***
****

## **Solution**
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: quote-service
  namespace: development
  labels:
    app: quote-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: quote-service
  template:
    metadata:
      labels:
        app: quote-service
    spec:
      containers:
      - name: quote-container
        image: datawire/quote:0.5.0
        ports:
          - containerPort: 8080
```

### Add resource to the POD. Use the below config under containers section.
```
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

<br/>

# Service
### A kubernetes service is a load balancer that directs traffic from the internet to kubernetes pods. A load balancer service has a public and static IP address.

* Service have permanent IP address.
* Static IP address that connect with each pod.

## Service Example

```
---
apiVersion: v1
kind: Service
metadata:
  name: demo-service
  namespace: development
spec:
  selector:
    app: pod-info
  ports:
    - port: 80
      targetPort: 3000
  type: LoadBalancer
```

### **Service Types**
* LoadBalancer
* NotePort
* ClusterIP

### **Create Service**
```
kubectl apply -f service.yaml
```

### **Get Services**
```
kubectl get services -n development
```

<br/>

# Kubernetes Architecture

### Kubernetes like a air port.
* **Cluster - Instance of kubernetes**
* **Control Plan**
* **Worker Node**

![k8s control plane](/learning-kubernetes/kubernetes-cluster.png?raw=true "kubernetes")

<br/>

## **Control Plane**

### If kubernetes like an airport then the control plane is like the air traffic control tower. So with it the people overlooking the cluster and making sure nodes and pods are created, modified, and delete without any issue.
![k8s control plane](/learning-kubernetes/Control-Plane.png?raw=true "kubernetes")

<br/>

## **Control Plane Component**

* **API Server**
* **ETCD**
* **Scheduler**
* **Controller Manager**
* **Cloud Controller Manager**

<br>

## API Server

Its cluster gateway that get any request from client for the cluster and its one entry point to the cluster.

* Acts as a gatekeeper for authentication.
* Verify the request that pass to pod and others.
* Each Pods, deployments, and autoscaler have API endpoints.
* Kubernetes API has a REST interface.
* Kubernetes API communicate via HTTP requests by using kubectl and kubeadm CLI tools.

```
kubectl api-resources
```
### Kube API Server is containerized application that run as a POD
```
kubectl get pods -n kube-system
```

## ETCD

Etcd the cluster brain and it is an open-source, highly avaiable key value store. In a kubernetes cluster it saves all data about the state of the cluster.

* Only the kube API server can communicate directly with etcd.
* Any of the cluster changes get stored in the key value store.

### Etcd is also run as a pod and can look at the logs to know how etcd works.
```
kubectl logs etcd-minikube -n kube-system
```

<br/>

## Scheduler

Scheduler identifies newly created pods that have not been assinge a worker node and then chooses a node for the pod to run on it.

* Schedule newly created pod.
* Identify and decides where to put the pod.
* Look at the nodes and put the pod where node resource are available.

<br/>

## Controller Manager

The controller manger is a loop that runs continually and check the status of the cluster to make sure things are running properly.

* Check the all worker nodes are up and running.
* It will remove the broken node.
* Replace the broken node with a new worker node.
* Looking POD crashing.
* Detects cluster state changes.

<br/>

## Cloud controller manager

Its connect the cluster with a cloud providers API so that can use cloud resources from AWS, Digital Ocean, Azure or any public cloud.

<br/>

## **Worker Node**
### The worker nodes are where pods are scheduled and run.
### If kubernetes like an airport then the control plane is like the air traffic control tower. Then the worker nodes are like the busy terminals, where planes park and passengers board.

![k8s worker nodes](/learning-kubernetes/Worker-nodes.png?raw=true)

## Worker Node Component
### In highly available most kubernets cluster run with a minimum of three worker nodes and each worker node has three component.

* **Kubelet**
* **Container Runtime**
* **Kube-Proxy**

<br/>

## Kubelet
### An agent that runs on every worker node.

* It make sure that containers in a pod are running and healthy.
* Communicates directly with the api-server in the control plane.
* It is looking for newly assinged pods.
* It interacts with both container and node.

<br/>

## Container Runtime
### Once the kubelet has been assigned a new pod, it starts the containers using the container runtime interface or CRI. CRI enables the kubelet to create containers with the engines.

* ContainerD
* CRI-O
* kata Containers
* AWS Firecracker

<br/>

## Kube-Proxy
* **Makes sure pods and services can communicate with other pods and services on nodes and in the control plane.**
* **Each kube-proxy communicates directly with the kube-apiserver**

<br/>

## How to work the control plane and nodes together
![control plane and nodes time series diagram](/learning-kubernetes/k8s-timeseries-diagram.png)

* A user run a kubernetes command for deployment. kubernetes config (kubeconfig) file gives permission to kubectl to communicate with the kubernetes cluster. kubectl command sends the information to the kube API server.

* Kube API server save the new deployment spec in etcd.

* While this is happening, the kube controller manager is checking the kube API server see if there have been any changes since its last loop.

* The kube scheduler then checks the kube API server to see there are any new pods that have not been assigned a node

* The kube API servel tell the scheduler that has not been placed on a specific node.

* So the scheduler chooses a node for the pod and sends that information back to the kube API server.

* The API server saves the state of the cluster in etcd.

* kubelet checks the API server to see if there are any new pods that it has been assigned.

* The kube API server sends the pod spec for the new pod to the kubelet.

* Then kubelet pulls the image and creates the container using the container runtime.

* The kubelet sends the pod status, healty or unhealthy back to the API server.

* Then API server saves the state to etcd.


<br>
<br>

# **Helm**
### Helm is the package management system for the kubernetes. Helm installs charts into Kubernetes, creating a new release for each installation. And to find new charts, you can search Helm chart repositories.

<br>

## Install Helm Chart
Let's install a helm chart called kube-state-metrics that help us to monitor kubernetes cluster.

### First add Helm repo
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

### Then update the Helm repo
```
helm repo update
```

### Show the Helm repo list
```
helm repo list
```

### Create a namespace for install the chart
```
kubectl create ns metrics
```

### Install the Helm chart
```
helm install kube-state-metrics bitnami/kube-state-metrics -n metrics
```

### See the metrics info
```
kubectl port-forward svc/kube-state-metrics 8080:8080 -n metrics
```

### Show the Helm chart
```
helm show chart bitnami/kube-state-metrics
```

### Show the Helm chart values
```
helm show values bitnami/kube-state-metrics
OR
helm show values bitnami/kube-state-metrics > values.yaml
```

### Show Helm release or list
```
helm list -n metrics
OR
helm ls -n metrics
```

### Helm upgrade chart version
```
helm upgrade kube-state-metrics bitnami/kube-state-metrics --version 3.3.3 -n metrics
```

### Helm delete release
```
helm delete kube-state-metrics -n metrics
```

<br>

## **Challenge**: Install a Helm Chart
* **Create a new namespace called metrics**
* **Install the bitnami hosted metrics server helm chart**
* **Ensure the release name is logs-metrics-server**
* **Enable the metrics API with the flag --set apiService.create=true**

<br>

# Create Helm Chart
Creat a own helm chart using command line like others.

### First make a directory
```
mkdir helm-course
cd helm-course/
```

### Create a Chart
```
helm create first-chart
ls first-chart/
```

### Create configmap for helm chart
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: first-chart-configmap
data:
  port: "8080"
```

### Insatll Helm Chart
```
helm install first-chart .
```

### Get configmap
```
kubectl get cm
```

### Describe configmap
```
kubectl describe cm first-chart-configmap
```

### See Helm changes on localy
```
helm template first-chart .
```

### Upgrade Helm chart
```
helm upgrade first-chart .
```

### Create secret for helm chart
```
apiVersion: v1
kind: Secret
metadata:
  name: first-secret
type: Opaque
data:
  username: YWRtaW4=
  password: c2VjcmV0
```

### Get kubernetes secrets
```
kubectl get secrets
```

### Describe kubernetes secrets
```
kubectl describe secret first-secret
```

### Show Secrets
```
kubectl get secret <secret-name> -o yaml
kubectl get secret <secret-name> -o jsonpath="{.data.DB_PASSWORD}" | base64 --decode
```

## Helm Rollback
### Get Helm history/releases
```
helm history first-chart
```

### Rollback Helm Chart
```
helm rollback first-chart
```


### Here’s a step-by-step guide to verify, test, and deploy your Helm charts on Minikube:

### Start Minikube
```
minikube start
```
#### Set Your Kubernetes Context to Minikube
```
kubectl config use-context minikube
```
#### Lint (Verify) Your Helm Chart
```
helm lint ./charts/my_service
```
This checks your chart for common errors.

#### Test Rendering with Your Values
```
helm template adc ./charts/my_service -f helmvars/dev/values.yaml
```
This shows what will be deployed, using your values.

### Run Helm Chart Tests (if you have test hooks defined)
```
helm test adc
```
(Only works after deployment and if you have test hooks in your chart.)

#### Decrypt the SOPS file
```
sops -d helmvars/dev/secrets.yaml > helmvars/dev/secrets.dec.yaml
```

#### Deploy Your Chart to Minikube
```
helm upgrade --install my_app ./charts/my_service \
 -f helmvars/dev/values.yaml \
-f helmvars/dev/sercret.dec.yaml
```

```
helm upgrade --install my-release ./charts/my_service \
  -f helmvars/dev/values.yaml \
  -f <(sops -d helmvars/dev/secrets.yaml)
```
my_app is the release name (you can choose any name).
Adjust the values file as needed for your environment.
#### Check Deployment Status
```
kubectl get pods
kubectl get svc
helm list
```
#### (Optional) Access Your App
If your app exposes a service, you can access it via Minikube:
```
minikube service <service-name>
```

#### (Optional) Clean Up
```
helm uninstall adc
minikube delete
```

####Summary:

Use `helm lint` to verify.
Use `helm template` to test rendering.
Use `helm upgrade --install` to deploy.
Use `kubectl` and `minikube` service to check and access your app.

#### Tell your shell to use Minikube’s Docker daemon
This makes Docker build directly inside Minikube’s internal Docker engine, so the image becomes available to Kubernetes without pushing it to a registry.
```
eval $(minikube docker-env)
```

#### Reset back to normal Docker after
```
eval $(minikube docker-env --unset)
```

