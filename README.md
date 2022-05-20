# sig-cloud-native

Welcome to SIG-CloudNative of SphereEx!

## Get started

All you have to do is to clone this repo and get your hands dirty with those examples!

## Elementary Tasks

### 0. Create your own namespace

Update the Namespace manifest `namespace.yaml` in `example` directory, substitute ${YOUR_NAMESPACE} with your namespace 

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ${YOUR_NAMESPACE}
```

 ```shell
 kubectl create -f example/namespace.yaml
 ```

 Expected results:

 ```shell
namespace/${YOUR_NAMESPACE} created
 ```

### 1. Deploy your own Nginx with using Deployment.

Update the Deployment manifest in `example` directory, substitute ${YOUR_NAMESPACE} with your namespace, choose `nginx:1.14.2` as application image:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
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

Expected results:

```shell
NAME                     READY   STATUS    RESTARTS   AGE
nginx-66b6c48dd5-4xpnf   1/1     Running   0          27s
nginx-66b6c48dd5-95955   1/1     Running   0          27s
nginx-66b6c48dd5-jk7fk   1/1     Running   0          27s
```

### 2. Access your Nginx from your laptop.

With the help of `Port-Forward` of Kubenetes, you can access to your Nginx instance very easily.

```shell
kubectl port-forward pod/nginx-66b6c48dd5-jk7fk 80:80 -n ${YOUR_NAMESPACE}
```

Expected results:

```shell
Forwarding from 127.0.0.1:80 -> 80
Forwarding from [::1]:80 -> 80
```

Or just `curl` this endpoint:

```shell
curl localhost
```

Expected results:

```shell
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

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
