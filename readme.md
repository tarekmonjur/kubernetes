
# Kubernetes
### kubernetes known as K8s develop by Google at 2014 based on GoLangs

* High availability and No downtime
* Scalability and High performance
* Recovery, backup and Restore

<br/>

## `Container`
#
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

## `Image`
#
A container image is a file with executable code that can be run as a continer.

<br/>

## `K8s Basic Components`
#

* POD
* Service
* Ingress
* Namespace
* External Configuration
* Volumes / Storage
* Replica
* Deployment

<br/>

## `POD`
#
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
## **`Challenge:`** Create your own development
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

## **`Solution`**
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

## `Service Example`
#
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

# Kubernetes Architecture

* **Cluster - Instance of kubernetes**
* **Control Plan**
* **Worker Node**

## **`Control Plan Component or Processes`**
***
![Alt text](/relative/path/to/img.jpg?raw=true "Optional Title")

