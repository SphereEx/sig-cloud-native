# sig-cloud-native

Welcome to SIG-CloudNative of SphereEx!

## Get started

All you have to do is to clone this repo and get your hands dirty with those examples!

Elementary Tasks:

- [0. Create your own namespace](https://github.com/SphereEx/sig-cloud-native#0-create-your-own-namespace)
- [1. Deploy your own Nginx with using Deployment](https://github.com/SphereEx/sig-cloud-native#1-deploy-your-own-nginx-with-using-deployment)
- [2. Access your Nginx from your laptop](https://github.com/SphereEx/sig-cloud-native#2-access-your-nginx-from-your-laptop)
- [3. Do some scaling to your Nginx](https://github.com/SphereEx/sig-cloud-native#3-do-some-scaling-to-your-nginx)
- [4. How to custom index.html with ConfigMap ?](https://github.com/SphereEx/sig-cloud-native#4-how-to-custom-indexhtml-with-configmap-)
- [5. How to avoid the problem when accessing the specific Pod ? ](https://github.com/SphereEx/sig-cloud-native#5-how-to-avoid-the-problem-when-accessing-the-specific-pod-) 
- [6. Try Loadbalance and Ingress](https://github.com/SphereEx/sig-cloud-native#6-try-loadbalance-and-ingress)

Advanced Tasks

- [1. Deploy a MySQL instance](https://github.com/SphereEx/sig-cloud-native#1-deploy-a-mysql-instance)
- [2. Deploy a Zookeeper Cluster](https://github.com/SphereEx/sig-cloud-native#2-deploy-a-zookeeper-cluster)
- [3. Deploy a ShardingSphere Proxy Cluster](https://github.com/SphereEx/sig-cloud-native#3-deploy-a-shardingsphere-proxy-cluster)
- [4. Using `Helm` to setup ShardingSphere Proxy Cluster](https://github.com/SphereEx/sig-cloud-native#4-using-helm-to-setup-shardingsphere-proxy-cluster)
- [5. Using `Operator` to setup ShardingSphere Proxy Cluster](https://github.com/SphereEx/sig-cloud-native#5-using-operator-to-setup-shardingsphere-proxy-cluster)

Ultimate tasks

- [1. Find the fragile parts of the deployment pattern](https://github.com/SphereEx/sig-cloud-native#1-find-the-fragile-parts-of-the-deployment-pattern)
- [2. How to adopt the cloud native smell ? ](https://github.com/SphereEx/sig-cloud-native#2-how-to-adopt-the-cloud-native-smell-)


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

`kubectl` provides convienent subcommand to scale your Nginx:

Scale to zero replicas and no incoming request will be received:

```shell
kubectl scale deployment nginx --replicas=0 -n ${YOUR_NAMESPACE} 
```

Expected results:

```shell
deployment.apps/nginx scaled
```

When get Pods with `kubectl get pods -n ${YOUR_NAMESPACE}`, you can see that:

```shell
No resources found in ${YOUR_NAMESPACE} namespace.
```

Scale with more replicas in case of any traffic spikes:
```shell
kubectl scale deployment nginx --replicas=5 -n ${YOUR_NAMESPACE}
```

Expected results:

```shell
NAME                     READY   STATUS    RESTARTS   AGE
nginx-66b6c48dd5-4tbg2   1/1     Running   0          36s
nginx-66b6c48dd5-5jgtl   1/1     Running   0          36s
nginx-66b6c48dd5-ctxr8   1/1     Running   0          36s
nginx-66b6c48dd5-jhszm   1/1     Running   0          36s
nginx-66b6c48dd5-vqkv8   1/1     Running   0          36s
```

### 4. How to custom index.html with ConfigMap ?

If you want to have your name on the page `index.html`, but adding this file to the Docker image is always not allowed. So you have to utilize `ConfigMap` to finish this:

Create a `index.html` of you own:

```shell
<!DOCTYPE html>
<html>
<head>
<title>Welcome ${YOUR_NAME}!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome ${YOUR_NAME}!</h1>
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

Create a `ConfigMap` to have this file:

```shell
kubectl create configmap my-index --from-file index.html -n ${YOUR_NAMESPACE} 
```

Check `ConfigMap` with `kubectl get configmap -n ${YOUR_NAMESPACE}`: 

```shell
NAME               DATA   AGE
my-index           1      3m51s
```

Review the resource content using `kubectl get configmap -n ${YOUR_NAMESPACE} -o yaml`

```yaml
apiVersion: v1
data:
  index.html: |
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome ${YOUR_NAME}!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome ${YOUR_NAME}!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>

    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>

    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
kind: ConfigMap
metadata:
  creationTimestamp: "2022-05-20T03:25:23Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:index.html: {}
    manager: kubectl
    operation: Update
    time: "2022-05-20T03:25:23Z"
  name: my-index
  namespace: ${YOUR_NAMESPACE} 
  resourceVersion: "17538160"
  uid: cfa8c683-6eb1-4c2f-8420-7cd806c2ee34
```

Mount this ConfigMap to your Nginx:

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
      # Claim a new volume
      volumes:
      - name: index
        configMap:
          name: my-index
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        # Mount volume
        volumeMounts:
        - name: index
          mountPath: /usr/share/nginx/html/
```

After all Pods have been updated, then `curl localhost` you can see the expected page.

### 5. How to avoid the problem when accessing the specific Pod ?  

TGIService! 

You can bind a `Service` to have loadbalance and readiness probe to your Nginx!

Using `selector` to match the labels of Nginx `Pods`, `Service` treat them as traffic endpoints:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: ${YOUR_NAMESPACE}
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - name: http
    protocol: TCP
    port: 8080
    targetPort: 80
```

Create a `Service` for your Nginx:

```shell
kubectl create -f example/service.yaml
```

Check accessbility of your Service:
```shell
kubectl port-forward svc/nginx 8080:8080 -n ${YOUR_NAMESPACE} 
```

Expected results:

```shell
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80
```
Execute `curl localhost:8080` to reach to Nginx through this `Service`:

```shell
<!DOCTYPE html>
<html>
<head>
<title>Welcome ${YOUR_NAME}!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome ${YOUR_NAME}!</h1>
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

### 6. Try Loadbalance and Ingress.

Service have serveral types, one of them is `LoadBalancer`, which means the `Nginx` service will be exposed with a public endpoints, such as EIP, ALB, depends on different cloud vendor.To update a existed workload with a manifest file, you need to use `kubectl apply` instead of `kubectl create`. 

Update the `type` property in `spec` of Nginx `Service` from `ClusterIP` to `LoadBalancer`: 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: ${YOUR_NAMESPACE}
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - name: http
    protocol: TCP
    port: 8080
    targetPort: 8
```

Then you can get the external address:
```shell
NAME    TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
nginx   LoadBalancer   172.17.172.227   106.75.26.68   8080:32478/TCP   62m
```

Now the `106.75.26.68` the public IP address of your application Nginx.



To have better `Ingress` experience, you need a AWS EKS cluster or GCP GKE cluster for test.


## Advanced Tasks

### 1. Deploy a MySQL instance.

### 2. Deploy a Zookeeper Cluster.

### 3. Deploy a ShardingSphere Proxy Cluster.

### 4. Using `Helm` to setup ShardingSphere Proxy Cluster.

### 5. Using `Operator` to setup ShardingSphere Proxy Cluster.

## Ultimate tasks

### 1. Find the fragile parts of the deployment pattern.

### 2. How to adopt the cloud native smell ? 
