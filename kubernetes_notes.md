
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
kubectl logs -l app=your-app-label				# Get logs for all Pods in a Deployment (useful for CI/CD debugging).
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
kubectl port-forward pods/{pod_name} 8000:80                     	# Forwards pod port 80 to localhost port 8000.
kubectl port-forward --address 0.0.0.0 pods/{pod_name} 8000:8000 	# Enables external access to the port.
kubectl expose pods/{pod_name} --port 9000,9001                  	# Exposes a pod on specified ports.
kubectl describe svc {service_name}                              	# Displays details about a service.
kubectl expose deployment {deployment_name}                      	# Exposes a deployment as a service.
--port 9000                                                      	# Host port.
--target-port 8000                                               	# Application port within the container.
--type ClusterIP/NodePort/LoadBalancer                           	# Specifies the service type.
```

---

## Kubernetes Labels

```bash
kubectl get all --show-labels                                  		# Lists objects with their labels.
kubectl label pods/{pod_name} org=shmu                         		# Adds or updates a label on a pod.
kubectl delete deployments,services --selector environment=dev 		# Deletes objects with a specific label.
```

---

## Kubernetes Namespaces

```bash
kubectl get namespaces                                            	# Lists all namespaces.
kubectl create namespace {namespace_name}                        	# Creates a new namespace.
kubectl config set-context --current --namespace {namespace_name} 	# Switches to a namespace.
~/.kube/config                                                    	# Path to the kubeconfig file for manual edits.
kubectl delete namespace {namespace_name}                         	# Deletes a namespace.
```

---

## Kubernetes Manifest

```bash
kubectl apply -f manifest.yaml                                                # Applies a manifest to create resources.
--selector env=prod,customer=custA                                            # Applies changes only to objects matching specific labels. **If I have different deployments with different labels in one manifest.**
--dry-run=server
kubectl apply --prune -f manifests/ --selector app=my-app 					  # Delete all resoucres with label my-app which are not declared on manifest.
kubectl set env deployment/{deployment_name} {VARIABLE_NAME}={VARIABLE_VALUE} # Sets environment variables.
```

---

## Kubernetes Rollback

```bash
kubectl rollout history deployment/my-app
kubectl rollout undo deployment/my-app
kubectl rollout undo deployment/my-app --to-revision=2
kubectl rollout status deployment/my-app
```

---

## Kubernetes ConfigMap

### Creating a ConfigMap

```bash
kubectl create configmap {configmap_name} \                # Creates a ConfigMap with specified variables.
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

**Base64 Encoding for Secrets**

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

## Creating and maintaining Cluster.
ADD
## Kubernetes Gitlab connect to cluster

```
You need kubeconfig file and store it as variable in GITLAB $KUBE_CONFIG.
As Text:
$KUBE_CONFIG - Text Variable in Gitlab, Masked
If it was decoded: echo "$KUBE_CONFIG" | base64 -d > ~/.kube/config
```

Gitlab (AS TEXT):

```yaml
stages:
  - deploy

deploy_job:
  stage: deploy
  image: alpine/k8s:1.24.1
  script:
    - echo "$KUBE_CONFIG" > /tmp/kubeconfig
    - export KUBECONFIG=/tmp/kubeconfig
    - kubectl config get-contexts
    - kubectl apply -f deployment.yaml
    - rm /tmp/kubeconfig
```

```
You need kubeconfig file and store it as variable in GITLAB $KUBE_CONFIG.
As File:
$KUBE_CONFIG - File Variable in Gitlab, Masked
```

Gitlab (AS FILE):

```yaml
stages:
  - deploy

deploy_job:
  stage: deploy
  image: alpine/k8s:1.24.1 # Prípadne iný obraz s kubectl
  variables:
    KUBECONFIG: $KUBE_CONFIG
  script:
    - kubectl config get-contexts
    - kubectl apply -f deployment.yaml
```

---

## Kubernetes Gitlab Using Private Repository
$DOCKER_SERVER, USER etc. are saved in Gitlab CI/CD variables.

```bash
- kubectl create secret docker-registry private-registry-secret \
--docker-server=$DOCKER_SERVER \
--docker-username=$DOCKER_USER \
--docker-password=$DOCKER_PASSWORD 
```

If we have "$DOCKER_CONFIG_JSON” variable in Gitlab
Example:
```json
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "dGhpcyBpcyBhIHNhbXBsZSBhdXRoIHN0cmluZw==",
      "email": "jan.novak@example.com"
    }
  }
}
```json

```bash
- echo "$DOCKER_CONFIG_JSON" > docker-config.json
- kubectl create secret generic my-private-registry-secret \
--from-file=.dockerconfigjson=docker-config.json \
--type=kubernetes.io/dockerconfigjson \
```
Manifest Example:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-private-registry-secret
  namespace: <tvoj-namespace>
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: "$DOCKER_AUTH_JSON"
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      imagePullSecrets:
      - name: my-private-registry-secret
      containers:
      - name: my-app-container
        image: moje-meno-pouzivatela/moj-sukromny-obraz:latest
        ports:
        - containerPort: 80
```

## Kubernetes Deployment

## Kubernetes Services

```bash
kubectl port-forward pods/{pod_name} 8000:80                     	# Forwards pod port 80 to localhost port 8000.
kubectl port-forward --address 0.0.0.0 pods/{pod_name} 8000:8000 	# Enables external access to the port.
kubectl expose pods/{pod_name} --port 9000,9001                  	# Exposes a pod on specified ports.
kubectl describe svc {service_name}                              	# Displays details about a service.
kubectl expose deployment {deployment_name}                      	# Exposes a deployment as a service.
--port 9000                                                      	# Host port.
--target-port 8000                                               	# Application port within the container.
--type ClusterIP/NodePort/LoadBalancer                           	# Specifies the service type.
```
---
```
TargetPort is what is your application listening what is coded inside the POD on your image.
No TargetPort = Port
```

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

```
http://<service-name>.<namespace>.svc.cluster.local:8080/api/data - DNS name for internal communication (different namespaces)
http://<service-name>:8080/api/data - DNS name for internal communication (same namespace)
```
```
<service-name>: This is the metadata.name you define in your Service YAML manifest. For example, in our previous example, the service name was backend-service.
<namespace>: This is the name of the namespace the service is in. For a service in the default namespace, the name would be default. For a service in the backend-ns namespace, it would be backend-ns.
svc: This is a required part of the FQDN and stands for "service". It tells the DNS resolver that this is a query for a Kubernetes service.
cluster.local: This is the default cluster domain. It's automatically configured for your cluster and is the root of the Kubernetes DNS hierarchy.
8080/api/data: Your port and path, optional. 
```

NodePort:

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
```
The "Port" part: A specific port is opened on the host (the node).
The "Node" part: This port is open on every node, regardless of whether a pod for that service is running on that particular node.
Port Range: The NodePort must be in a specific, predefined range. By default, this range is 30000-32767.
Exposed on All Nodes: This is a major point of consideration. The port is open on every node's IP address. This means you need to be careful with your network security and firewall rules.
```

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
```
External IP from cloud provider. Without external IP it still works internally.
```

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
```
What it is: An ExternalName service is a non-proxying service. It doesn't have a ClusterIP, and it doesn't select pods. Instead, it creates a DNS CNAME record that points to an external FQDN.
How it works: When a client pod tries to access an ExternalName service, the DNS query is resolved directly to the external hostname. Kubernetes doesn't forward any traffic; it just provides a DNS alias.
```

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
```
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
```
A request to http://myapp.example.com/api/users will be routed to the api-service.
A request to http://myapp.example.com/ will be routed to the frontend-service.
A request to http://myapp.example.com/about will also be routed to the frontend-service because / is a prefix match for everything that doesn't match the more specific /api.
```
pathType is a key field that determines how the path is matched. There are three options:
Prefix (Most Common):
```
Behavior: Matches based on a URL path prefix. The path must be a prefix of the URL path being requested.
Example:
path: /api will match /api, /api/v1, and /api/users.
path: / will match / and any other path that starts with /.
Note: When using multiple Prefix paths, the most specific one is always chosen. The Ingress Controller will try to match the longest path first.
```
Exact:
```
Behavior: Matches the URL path exactly, case-sensitively.
Example:
path: /contact will only match /contact. It will not match /contact/us.
Use case: Use this when you have a specific, singular endpoint that you want to route.
```
---

### Creating and using ConfigMap in a Manifest

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  database_url: "jdbc:postgresql://postgres-service:5432/myapp"
  log_level: "INFO"
  app_settings: | # multi-line example
    server:
      port: 8080
      timeout: 30s
    features:
      enable_beta_feature: false
```

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


```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials-plain
  namespace: default
type: Opaque
stringData:
  username: admin
  password: s3cr3t_p@ss
```
---

## Executing Commands on Startup or Before Shutdown

Kubernetes has a few different ways to run commands within a container at key moments.
The most common way is to define the entry point and arguments for your container directly in your Deployment manifest.
```
command: This overrides the default ENTRYPOINT in your Docker image. It's the primary command that will be executed when the container starts.
args: These are the arguments passed to the command.
```
Example:
If your Docker image doesn't have a default entry point, you could use command to start a shell and then run your script.
YAML

```yaml
spec:
  containers:
  - name: my-app
    image: my-app:latest
    command: ["/bin/sh", "-c"] # Overrides default entrypoint
    args: ["/app/startup.sh"]   # Passes a script as an argument
```

---
## Lifecycle hooks

Lifecycle hooks let you run a command after a container is created or before it's terminated.
```
postStart Hook: This hook is executed immediately after a container is created. It's useful for running initial setup commands that your application needs before it can start serving traffic. For example, you could run a database migration script.
preStop Hook: This hook is executed just before a container is terminated. It's perfect for a graceful shutdown, like draining connections or cleaning up temporary files.
```
Example:
This manifest uses a postStart hook to run a script and a preStop hook to send a termination signal.
YAML

```yaml
spec:
  containers:
  - name: my-app
    image: my-app:latest
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh", "-c", "echo 'Container started, running setup script.' && /app/setup-database.sh"]
      preStop:
        exec:
          command: ["/bin/sh", "-c", "echo 'Shutting down gracefully...' && sleep 10"]
```

---

## Verifying Readiness with Probes

You can also use probes to execute commands and verify that your application is running correctly. Probes check the state of your application and can affect how Kubernetes manages traffic and restarts.
This probe checks if your container is ready to accept incoming traffic. If a readinessProbe fails, Kubernetes won't send requests to the container.
```
How it works: You can define a command that Kubernetes will execute periodically. If the command exits with a status code of 0, the container is considered ready. If it returns a non-zero code, it's not ready.
```
Example:
This probe checks if a file exists, indicating that the application is ready.
YAML

```yaml
spec:
  containers:
  - name: my-app
    image: my-app:latest
    readinessProbe:
      exec:
        command: ["cat", "/tmp/ready.txt"]
      initialDelaySeconds: 5
      periodSeconds: 5
```

```yaml
spec:
  containers:
  - name: my-app
    image: my-app:latest
    livenessProbe:
    	httpGet:
    		path: /healthz
            port: 8080
    	initialDelaySeconds: 15
    	periodSeconds: 20
```yaml
