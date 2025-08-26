
# Kubernetes Notes

## Table of Contents
1. [General Commands](#general-commands)
2. [Kubernetes Pods](#kubernetes-pods)
3. [Kubernetes Deployments](#kubernetes-deployments)
4. [Kubernetes Services](#kubernetes-services)
5. [Kubernetes Labels](#kubernetes-labels)
6. [Kubernetes Namespaces](#kubernetes-namespaces)
7. [Kubernetes Manifest, Variables, and Secrets](#kubernetes-manifest)
8. [Kubernetes ConfigMap](#kubernetes-configmap)
9. [Kubernetes Secrets](#kubernetes-secrets)
10. [Kubernetes Ingress](#kubernetes-ingress)
11. [Kubernetes Persistent Volumes and Claims (PV, PVC)](#kubernetes-persistent-volumes-and-claims-pv-pvc)
12. [Examples](#examples)

---

## General Commands

```bash
kubectl version                            # Shows installed version of kubectl.
kubectl cluster-info                       # Displays cluster information.
kubectl config view --raw                  # Displays the raw configuration file.
```

---

## Kubernetes Pods

```bash
kubectl run {pod_name} --image {image_name}     # Creates a pod with the specified name and image.
--dry-run=client                                # Tests the command without actually creating the pod.
--output yaml                                   # Outputs the command as a YAML file.
kubectl get pods                                # Lists all pods.
kubectl get all                                 # Lists all Kubernetes objects.
--namespace {namespace_name}                    # Filters objects in a specific namespace.
kubectl logs pods/{pod_name} --follow --tail 1  # Displays live logs (last line) of a pod.
kubectl describe pods/{pod_name}                # Shows detailed information about a pod.
kubectl get pods --watch                        # Monitors pod creation and deletion live.
kubectl exec pods/{pod_name} -- hostname        # Executes a command inside the pod (e.g., `hostname`).
kubectl exec -it pods/{pod_name} -- bash        # Opens an interactive bash shell inside the pod.
```

---

## Kubernetes Deployments

```bash
kubectl create deployment {deployment_name} --image {image_name}               # Creates a deployment from an image.
--replicas 5                                                                   # Specifies the number of replicas.
kubectl delete deployments/{deployment_name}                                   # Deletes a specific deployment.
kubectl delete deployments --all                                               # Deletes all deployments.
kubectl scale deploy/{deployment_name} --replicas 2                            # Scales a deployment to 2 replicas.
kubectl set image deployments/{deployment_name} {deployment_name}={image_name} # Updates the deployment's image.
```

---

## Kubernetes Services

```bash
kubectl port-forward pods/{pod_name} 8000:80                     # Forwards pod port 80 to localhost port 8000.
kubectl port-forward --address 0.0.0.0 pods/{pod_name} 8000:8000 # Enables external access to the port.
kubectl expose pods/{pod_name} --port 9000,9001                  # Exposes a pod on specified ports.
kubectl describe svc {service_name}                              # Displays details about a service.
kubectl expose deployment {deployment_name}                      # Exposes a deployment as a service.
--port 9000                                                      # Host port.
--target-port 8000                                               # Application port within the container.
--type ClusterIP/NodePort/LoadBalancer                           # Specifies the service type.
```

---

## Kubernetes Labels

```bash
kubectl get all --show-labels                                  # Lists objects with their labels.
kubectl label pods/{pod_name} org=shmu                         # Adds or updates a label on a pod.
kubectl delete deployments,services --selector environment=dev # Deletes objects with a specific label.
```

---

## Kubernetes Namespaces

```bash
kubectl get namespaces                                            # Lists all namespaces.
kubectl create namespace {namespace_name}                         # Creates a new namespace.
kubectl config set-context --current --namespace {namespace_name} # Switches to a namespace.
~/.kube/config                                                    # Path to the kubeconfig file for manual edits.
kubectl delete namespace {namespace_name}                         # Deletes a namespace.
```

---

## Kubernetes Manifest

```bash
kubectl apply -f manifest.yaml                                                # Applies a manifest to create resources.
--selector env=prod,customer=custA                                            # Applies changes only to objects matching specific labels.
kubectl set env deployment/{deployment_name} {VARIABLE_NAME}={VARIABLE_VALUE} # Sets environment variables.
```
## Kubernetes Authentication

kube config for authentication service account

	vytvorit variable bud file kubeconfig cely alebo service account token protected variable

	before script
    - echo "$KUBE_CONFIG" | base64 -d > ~/.kube/config # $KUBE_CONFIG is a GitLab variable


MORE MORE MORE

## Kubernetes Services

TargetPort is what is your application listening what is coded inside the POD on your image.

No TargetPort = Port

ClusterIP:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  type: ClusterIP
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
  - name: metrics
    port: 9090
    targetPort: 9090
    protocol: TCP
```

http://<service-name>.<namespace>.svc.cluster.local:8080/api/data - DNS name for internal communication (different namespaces)
http://<service-name>:8080/api/data - DNS name for internal communication (same namespace)

<service-name>: This is the metadata.name you define in your Service YAML manifest. For example, in our previous example, the service name was backend-service.

<namespace>: This is the name of the namespace the service is in. For a service in the default namespace, the name would be default. For a service in the backend-ns namespace, it would be backend-ns.

svc: This is a required part of the FQDN and stands for "service". It tells the DNS resolver that this is a query for a Kubernetes service.

cluster.local: This is the default cluster domain. It's automatically configured for your cluster and is the root of the Kubernetes DNS hierarchy.

8080/api/data: Your port and path, optional. 

NodePort:

port.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-nodeport-service
spec:
  selector:
    app: my-app
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 30005 # Remove if you want auto assign of IP
```

The "Port" part: A specific port is opened on the host (the node).
The "Node" part: This port is open on every node, regardless of whether a pod for that service is running on that particular node.

Port Range: The NodePort must be in a specific, predefined range. By default, this range is 30000-32767.

Exposed on All Nodes: This is a major point of consideration. The port is open on every node's IP address. This means you need to be careful with your network security and firewall rules.

LoadBalancer:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-loadbalancer-service
spec:
  selector:
    app: my-app
  type: LoadBalancer
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
```

External IP from cloud provider. Without external IP it still works internally.

ExternalName:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-legacy-api
spec:
  type: ExternalName
  externalName: legacy-api.example.com
```

What it is: An ExternalName service is a non-proxying service. It doesn't have a ClusterIP, and it doesn't select pods. Instead, it creates a DNS CNAME record that points to an external FQDN.

How it works: When a client pod tries to access an ExternalName service, the DNS query is resolved directly to the external hostname. Kubernetes doesn't forward any traffic; it just provides a DNS alias.

Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app-service
            port:
              number: 80

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
spec:
  rules:
  - host: myapp.example.com #Use when you have DNS record.
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

With this configuration:

A request to http://myapp.example.com/api/users will be routed to the api-service.

A request to http://myapp.example.com/ will be routed to the frontend-service.

A request to http://myapp.example.com/about will also be routed to the frontend-service because / is a prefix match for everything that doesn't match the more specific /api.

pathType Options

pathType is a key field that determines how the path is matched. There are three options:

Prefix (Most Common):

Behavior: Matches based on a URL path prefix. The path must be a prefix of the URL path being requested.

Example:

path: /api will match /api, /api/v1, and /api/users.

path: / will match / and any other path that starts with /.

Note: When using multiple Prefix paths, the most specific one is always chosen. The Ingress Controller will try to match the longest path first.

Exact:

Behavior: Matches the URL path exactly, case-sensitively.

Example:

path: /contact will only match /contact. It will not match /contact/us.

Use case: Use this when you have a specific, singular endpoint that you want to route.

AUTHERNTICATION
kube config for authentication service account

	vytvorit variable bud file kubeconfig cely alebo service account token protected variable

	before script
    - echo "$KUBE_CONFIG" | base64 -d > ~/.kube/config # $KUBE_CONFIG is a GitLab variable







```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: minio
  name: minio
  namespace: storage
spec:
  replicas: 3
  selector:
    matchLabels:
      app: minio
  template:
    metadata:
      labels:
        app: minio
    spec:
      containers:
      - image: bitnami/minio
        name: minio
        env:
        - name: MINIO_DEFAULT_BUCKETS
          value: music,pictures,videos,books
        - secretRef:
          name: minio

---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: minio
  name: minio-cl
  namespace: storage
spec:
  ports:
  - name: port-1
    port: 9000
    protocol: TCP
    targetPort: 9000
  - name: port-2
    port: 9001
    protocol: TCP
    targetPort: 9001
  selector:
    app: minio

---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: minio
  name: minio-lb
  namespace: storage
spec:
  ports:
  - name: port-1
    port: 9000
    protocol: TCP
    targetPort: 9000
  - name: port-2
    port: 9001
    protocol: TCP
    targetPort: 9001
  selector:
    app: minio
  type: ClusterIP
  
---
  
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minio
  namespace: storage
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: minio-cl
            port:
              number: 9001
        path: /
        pathType: Prefix
```

---

## Kubernetes ConfigMap

### Creating a ConfigMap

```bash
kubectl create configmap {configmap_name} \                            # Creates a ConfigMap with specified variables.
  --from-literal=WEATHER_UPDATE_INTERVAL=30 \      
  --from-literal=WEATHER_UNITS=standard \
  --from-literal=WEATHER_QUERY=poprad,sk

kubectl create configmap {configmap_name} --from-env-file yourfile.env # Creates a ConfigMap from a file.
```

**Example `yourfile.env`**
```env
WEATHER_UPDATE_INTERVAL=30
WEATHER_UNITS=standard 
WEATHER_QUERY=poprad,sk
```

### Using a ConfigMap in a Manifest

```yaml
env:                                          # Sets individual variables.
- name: {VARIABLE_NAME}
  valueFrom:
    configMapKeyRef:
      name: {configmap_name}
      key: {VARIABLE_NAME}

envFrom:                                      # Loads all variables from a ConfigMap.
- configMapRef:
    name: {configmap_name}
```

```bash
kubectl get configmaps                        # Lists all ConfigMaps.
kubectl describe configmaps {configmap_name}  # Shows details of a specific ConfigMap.
kubectl delete configmap {configmap_name}     # Deletes a ConfigMap.
```

---

## Kubernetes Secrets

### Creating Secrets

```bash
kubectl create secret generic {secret_name} --from-literal {VARIABLE_NAME}={VARIABLE_VALUE} # Creates a secret with variables.
kubectl create secret generic {secret_name} --from-env-file yourfile.env                    # Creates a secret from a file.
```

### Managing Secrets

```bash
kubectl get secrets                            # Lists all secrets.
kubectl describe secrets {secret_name}         # Shows details of a specific secret.
```

- **Base64 Encoding for Secrets**:
  ```bash
  echo -n '{YOUR_VALUE}' | base64                   # Encodes a value to Base64.
  echo -n '{YOUR_VALUE}' | base64 | base64 --decode # Decodes a Base64 value.
  ```

---

## Kubernetes Ingress

```bash
kubectl create ingress weather --rule /=weather:8000 --rule /*=weather:8000

kubectl describe ingress weather --namespace weather # Describes ingress in a specific namespace.
kubectl delete ingress weather                       # Deletes an ingress resource.
```

---

## Kubernetes Persistent Volumes and Claims (PV, PVC)

```bash
kubectl get pv                                    # Lists Persistent Volumes (PVs).
kubectl get pvc                                   # Lists Persistent Volume Claims (PVCs).
```

---

**Refer to templates for specific examples.**

---

# Examples

`deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: deployment name

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
      - image: nginxdemos/hello
        name: container name
```

`service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - name: nginx-name
    port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer
```

---

`deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-pod
  name: nginx-pod
  namespace: dev-environment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
    spec:
      containers:
      - image: nginx:latest
        name: nginx-pod
        resources:
          requests:
            memory: "128Mi"
            cpu: "0.5"
          limits:
            memory: "128Mi"
            cpu: "1"
        volumeMounts:
        - name: nginx-config-volume
          mountPath: /usr/share/nginx/html
          readOnly: true
      volumes:
      - name: nginx-config-volume
        configMap:
          name: nginx-config
---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-pod
  name: nginx-service
  namespace: dev-environment
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
    nodePort: 30000
  selector:
    app: nginx-pod
  type: NodePort
```
