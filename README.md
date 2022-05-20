# sig-cloud-native

Welcome to SIG-CloudNative of SphereEx!

## Get started

All you have to do is to clone this repo and get your hands dirty with those examples!

## Elementary Tasks

### 1. Deploy your own Nginx with using Deployment.

Update the Deployment manifest in `example` directory, substitute ${YOUR_NAMESPACE} with your namespace, choose `nginx:1.14.2` as application image:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: ${YOUR_NAMESPACE}
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

Execute the following command: 

```shell
kubectl create -f example/deployment.yaml
```

Check the Pod status:

```shell
kubectl get pod -n ${YOUR_NAMESPACE}
```

### 2. Access your Nginx from your laptop.

### 3. Do some scaling to your Nginx.

### 4. How to custom index.html with ConfigMap ?

### 5. How to avoid the problem when accessing the specific Pod ?  

### 6. Try Loadbalance and Ingress.

## Advanced Tasks

### 1. Deploy a MySQL instance.

### 2. Deploy a Zookeeper Cluster.

### 3. Deploy a ShardingSphere Proxy Cluster.

### 4. Using `Helm` to setup ShardingSphere Proxy Cluster.

### 5. Using `Operator` to setup ShardingSphere Proxy Cluster.

## Ultimate tasks

### 1. Find the fragile parts of the deployment pattern.

### 2. How to adopt the cloud native smell ? 
