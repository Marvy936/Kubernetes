# Table of Contents

1. [Useful Commands](#useful-commands)
   - [General Commands](#general-commands)
   - [Kubernetes Pods](#kubernetes-pods)
   - [Kubernetes Deployments](#kubernetes-deployments)
   - [Kubernetes Services](#kubernetes-services)
   - [Kubernetes Labels](#kubernetes-labels)
   - [Kubernetes Namespaces](#kubernetes-namespaces)
   - [Kubernetes Manifest](#kubernetes-manifest)
   - [Kubernetes Rollback](#kubernetes-rollback)

2. [Kubernetes ConfigMap](#kubernetes-configmap)
3. [Kubernetes Secrets](#kubernetes-secrets)
4. [Creating and Maintaining Cluster](#creating-and-maintaining-cluster)
5. [Kubernetes GitLab Integration](#kubernetes-gitlab-integration)
   - [Using kubeconfig as Text Variable](#using-kubeconfig-as-text-variable)
   - [Using kubeconfig as File Variable](#using-kubeconfig-as-file-variable)
   - [Using Private Docker Registry](#using-private-docker-registry)

6. [Kubernetes Deployment Examples](#kubernetes-deployment-examples)
   - [Basic Deployment](#basic-deployment)
   - [Deployment with Resources and Autoscaling](#deployment-with-resources-and-autoscaling)
   - [Deployment with Init and Sidecar Containers](#deployment-with-init-and-sidecar-containers)
   - [Deployment with Probes (Liveness, Readiness, Startup)](#deployment-with-probes-liveness-readiness-startup)
   - [Deployment with ConfigMap & Secret Environment Variables](#deployment-with-configmap--secret-environment-variables)
   - [Deployment with Volumes (PVC, emptyDir)](#deployment-with-volumes-pvc-emptydir)
   - [Deployment with Security Context, NodeSelector, Affinity, Tolerations](#deployment-with-security-context-nodeselector-affinity-tolerations)
   - [Horizontal Pod Autoscaler Example](#horizontal-pod-autoscaler-example)

7. [Using GitLab Variables in Manifests](#using-gitlab-variables-in-manifests)
   - [Template Example](#template-example-deploymenttemplateyaml)
   - [GitLab CI/CD Integration](#gitlab-cicd-integration)

8. [Kubernetes Volumes](#kubernetes-volumes)
   - [Ephemeral Volumes](#ephemeral-volumes)
   - [Persistent Volumes](#persistent-volumes)
   - [StorageClass and Dynamic Provisioning](#storageclass-and-dynamic-provisioning)
   - [Overview of Supported Storage Backends](#overview-of-supported-storage-backends)
     - [Local](#local)
     - [Network](#network)
     - [Cloud](#cloud)

9. [Kubernetes Services](#kubernetes-services-1)
   - [ClusterIP](#clusterip)
   - [NodePort](#nodeport)
   - [LoadBalancer](#loadbalancer)
   - [ExternalName](#externalname)
   - [Ingress](#ingress)
     - [Path-Based Routing](#path-based-routing)
     - [PathType (Prefix, Exact)](#pathtype-prefix-exact)

10. [Executing Commands on Startup / Shutdown](#executing-commands-on-startup--shutdown)
    - [Command and Args](#command-and-args)
    - [Lifecycle Hooks (postStart, preStop)](#lifecycle-hooks-poststart-prestop)

11. [Verifying Readiness with Probes](#verifying-readiness-with-probes)
    - [Readiness Probe](#readiness-probe)
    - [Liveness Probe](#liveness-probe)

12. [Kubernetes StatefulSet](#kubernetes-statefulset)
    - [Headless Service](#headless-service)
    - [StatefulSet Manifest with PVCs](#statefulset-manifest-with-pvcs)

---

## Useful Commands
### General Commands

```bash
kubectl version                            # Shows installed version of kubectl.
kubectl cluster-info                       # Displays cluster information.
kubectl config view --raw                  # Displays the raw configuration file.
```

---

### Kubernetes Pods

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

### Kubernetes Deployments

```bash
kubectl create deployment {deployment_name} --image {image_name}               # Creates a deployment from an image.
--replicas 5                                                                   # Specifies the number of replicas.
kubectl delete deployments/{deployment_name}                                   # Deletes a specific deployment.
kubectl delete deployments --all                                               # Deletes all deployments.
kubectl scale deploy/{deployment_name} --replicas 2                            # Scales a deployment to 2 replicas.
kubectl set image deployments/{deployment_name} {deployment_name}={image_name} # Updates the deployment's image.
```

---

### Kubernetes Services

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

### Kubernetes Labels

```bash
kubectl get all --show-labels                                  		# Lists objects with their labels.
kubectl label pods/{pod_name} org=shmu                         		# Adds or updates a label on a pod.
kubectl delete deployments,services --selector environment=dev 		# Deletes objects with a specific label.
```

---

### Kubernetes Namespaces

```bash
kubectl get namespaces                                            	# Lists all namespaces.
kubectl create namespace {namespace_name}                        	# Creates a new namespace.
kubectl config set-context --current --namespace {namespace_name} 	# Switches to a namespace.
~/.kube/config                                                    	# Path to the kubeconfig file for manual edits.
kubectl delete namespace {namespace_name}                         	# Deletes a namespace.
```

---

### Kubernetes Manifest

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

### Kubernetes ConfigMap

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

### Kubernetes Secrets

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

### Kubernetes Ingress

```bash
kubectl create ingress weather --rule /=weather:8000 --rule /*=weather:8000

kubectl describe ingress weather --namespace weather # Describes ingress in a specific namespace.
kubectl delete ingress weather                       # Deletes an ingress resource.
```

---

### Kubernetes Persistent Volumes and Claims (PV, PVC)

```bash
kubectl get pv                                    # Lists Persistent Volumes (PVs).
kubectl get pvc                                   # Lists Persistent Volume Claims (PVCs).
```

---

## Creating and maintaining Cluster.

Scenario: You have two VMs in the cloud, and you want to set up a Kubernetes cluster on them. Since Kubernetes needs at least one control plane (master) node and one or more worker nodes, your two VMs can work like this:

VM1 → Control plane (master)
VM2 → Worker node

🔹 1. Prepare Both VMs

Run on both VM1 and VM2:
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Disable swap (required for Kubernetes)
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab

# Load required modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Configure networking
cat <<EOF | sudo tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

🔹 2. Install Container Runtime (Containerd recommended)

On both VMs:
```bash
sudo apt install -y containerd
```

Configure containerd:
```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd
```

🔹 3. Install kubeadm, kubelet, kubectl

On both VMs:
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | \
  sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /" | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

🔹 4. Initialize Control Plane (on VM1)

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

🔹 5. Install Network Plugin

For Flannel:
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
(You can also use Calico or Cilium, but Flannel is simplest.)

🔹 6. Join Worker Node (VM2)

On VM1 you’ll see a kubeadm join command at the end of kubeadm init output.
Something like:
```bash
kubeadm join <MASTER_IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
```
Run that command on VM2.
If you lost the token, regenerate it on VM1:
```bash
kubeadm token create --print-join-command
```

🔹 7. Verify Cluster
On VM1:
```bash
kubectl get nodes
```

You should see:

```
NAME     STATUS   ROLES           AGE   VERSION
vm1      Ready    control-plane   XXm   v1.31.x
vm2      Ready    <none>          XXm   v1.31.x
```

Networking and firewall:

🔹 Required Kubernetes Ports

(from Kubernetes official docs)

Between control plane (VM1) and workers (VM2):

```
6443/tcp → Kubernetes API server (all nodes need to reach this on the control-plane node).

10250/tcp → Kubelet API (control plane ↔ worker communication).

10259/tcp, 10257/tcp → kube-scheduler, kube-controller-manager (only needed if running multiple control-plane nodes, so for your 2-node setup not critical).

8472/udp → Flannel VXLAN (if you use Flannel networking).

179/tcp → (only if using Calico BGP).

4789/udp → (if using Cilium with VXLAN).

Node-to-node traffic (Pods, Services):

30000–32767/tcp → NodePort services (if you expose services via NodePort).

All traffic inside Pod network CIDR (e.g. 10.244.0.0/16 for Flannel) → must be allowed between VMs.
```

🔹 Cloud Firewall / Security Groups

In your cloud provider, check firewall rules or security groups.
You need at least:

VM1 (control plane) inbound:
```
6443/tcp open to VM2 (and your own IP if you want kubectl access).
```
VM1 ↔ VM2:
```
Allow pod CIDR traffic (default Flannel = 10.244.0.0/16).

Allow Kubernetes service CIDR traffic.

Allow required kubelet + overlay network ports (10250/tcp, 8472/udp, etc.).
```

🔹 Optional (for external access)

```
If you want to reach workloads from the internet:

Ingress controller (usually on ports 80/tcp and 443/tcp).

Or use a LoadBalancer service if your cloud provider supports it.

Or open NodePort range (30000–32767/tcp).
```

For running only single node cluster you have to remove taint from master node:

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

## Kubernetes GitLab Integration

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
```

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
    spec:                                  # This Part
      imagePullSecrets:                    # This Part
      - name: my-private-registry-secret   # This Part
      containers:
      - name: my-app-container
        image: moje-meno-pouzivatela/moj-sukromny-obraz:latest
        ports:
        - containerPort: 80
```

## Kubernetes Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-resources
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
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          requests:     # minimálne zdroje
            cpu: "100m"
            memory: "128Mi"
          limits:       # maximálne zdroje
            cpu: "500m"
            memory: "256Mi"
```
```yaml
#Funguje s deploymentom hore.
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-resources
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70   # keď CPU >70%, pridá repliky
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-big-demo
  labels:
    app: nginx
spec:
  replicas: 3   # ✅ počet replík
  strategy:     # ✅ stratégia nasadzovania
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1  # koľko podov môže byť naraz dole
      maxSurge: 1        # koľko nových podov môže pribudnúť

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx
      annotations:
        prometheus.io/scrape: "true"   # ✅ príklad pre monitoring
        prometheus.io/port: "80"

    spec:
      # ✅ Init container (beží pred hlavnými kontajnermi)
      initContainers:
      - name: init-myservice
        image: busybox
        command: ['sh', '-c', 'echo Initializing... && sleep 5']

      # ✅ Sidecar container (spolu s hlavným kontajnerom)
      containers:
      - name: nginx
        image: nginx:1.25
        imagePullPolicy: IfNotPresent  # Always/IfNotPresent/Never
        ports:
        - containerPort: 80

        # ✅ Health checks
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10

        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

        startupProbe:
          httpGet:
            path: /
            port: 80
          failureThreshold: 30
          periodSeconds: 10

        # ✅ Resources
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"

        # ✅ Env variables (ConfigMap + Secret)
        env:
        - name: WELCOME_MSG
          valueFrom:
            configMapKeyRef:
              name: nginx-config
              key: WELCOME_MSG
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: nginx-secret
              key: PASSWORD

        # ✅ Mount storage
        volumeMounts:
        - name: nginx-data
          mountPath: /usr/share/nginx/html

      # ✅ Sidecar – log collector
      - name: log-collector
        image: busybox
        command: ['sh', '-c', 'tail -n+1 -F /var/log/nginx/access.log']
        volumeMounts:
        - name: nginx-logs
          mountPath: /var/log/nginx

      # ✅ Security context na úrovni Podu
      securityContext:
        runAsUser: 1000
        runAsGroup: 3000
        fsGroup: 2000

      # ✅ Scheduler hints
      nodeSelector:
        disktype: ssd

      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: disktype
                operator: In
                values:
                - ssd

      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "nginx"
        effect: "NoSchedule"

      serviceAccountName: nginx-serviceaccount  # ✅ RBAC service account

      volumes:
      - name: nginx-data
        persistentVolumeClaim:
          claimName: pvc-nginx
      - name: nginx-logs
        emptyDir: {}   # logy sú len ephemeral

---
# ✅ ConfigMap (na env var)
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  WELCOME_MSG: "Hello from ConfigMap"

---
# ✅ Secret (na heslo)
apiVersion: v1
kind: Secret
metadata:
  name: nginx-secret
type: Opaque
data:
  PASSWORD: c2VjcmV0   # base64("secret")

---
# ✅ PersistentVolumeClaim (pre nginx html dáta)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nginx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard

---
# ✅ Horizontal Pod Autoscaler (HPA)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-big-demo
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75

```
## How to USE Gitlab Variables in manifest.

deployment.template.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $APP_NAME
spec:
  replicas: $REPLICAS
  template:
    metadata:
      labels:
        app: $APP_NAME
    spec:
      containers:
      - name: $APP_NAME
        image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```
gitlab-ci.yml
```yaml
deploy:
  stage: deploy
  script:
    - envsubst < k8s/deployment.yaml.template > k8s/deployment.yaml
    - kubectl apply -f k8s/deployment.yaml
```

## Kubernetes Volumes

1. Ephemeral (dočasné) volumes

Používajú sa, keď nepotrebuješ uchovať dáta po zániku Podu.
```
emptyDir – vytvorí prázdny adresár na node, zmizne po zmazaní Podu.
ephemeral volume (CSI inline) – krátkodobý volume od storage drivera (napr. pre testy).
```

emptyDir:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-emptydir
spec:
  containers:
  - name: busybox
    image: busybox
    command: [ "sleep", "3600" ]
    volumeMounts:
    - name: temp-storage
      mountPath: /tmp/data
  volumes:
  - name: temp-storage
    emptyDir: {}
```

2. Persistent Volumes (trvalé dáta)

Keď dáta musia prežiť reštart podu alebo jeho presun.
Používa sa kombinácia:
```
PersistentVolume (PV) – zdroj storage v clustri (napr. disk, NFS, cloud storage).
PersistentVolumeClaim (PVC) – žiadosť podu o konkrétny storage.
```
Pod takto nemusí vedieť detaily o tom, kde storage fyzicky je.
I have to create PV then Clam it with PVC and I have to mention storage under Pod.

hostPath:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data   # 📍 toto je cesta na NODE (host machine)
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-hostpath
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-hostpath
spec:
  containers:
  - name: app
    image: busybox
    command: [ "sleep", "3600" ]
    volumeMounts:
    - name: storage
      mountPath: /data   # 📍 toto je cesta VO VNÚTRI kontajnera
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: pvc-hostpath
```
NFS:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 10.0.0.100   # IP NFS servera
    path: /exports/data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-nfs
spec:
  containers:
  - name: app
    image: busybox
    command: [ "sleep", "3600" ]
    volumeMounts:
    - name: nfs-storage
      mountPath: /mnt/nfs
  volumes:
  - name: nfs-storage
    persistentVolumeClaim:
      claimName: pvc-nfs
```
StorageClass + Dynamic provisioning (cloud / CSI):
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-dynamic
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-dynamic
spec:
  containers:
  - name: app
    image: busybox
    command: [ "sleep", "3600" ]
    volumeMounts:
    - name: dynamic-storage
      mountPath: /data
  volumes:
  - name: dynamic-storage
    persistentVolumeClaim:
      claimName: pvc-dynamic
```

Kubernetes podporuje množstvo backendov (storage pluginov):

Lokálne riešenia
```
hostPath – mount lokálneho adresára z node (nevhodné pre produkciu).
local – lepšie riešenie, viaže PV na konkrétny node (pre high-performance storage).
```
Sieťové riešenia
```
NFS – klasický network filesystem.
iSCSI – block-level storage.
CephFS, RBD (Ceph block storage).
GlusterFS.
Fibre Channel.
```
Cloud storage (väčšinou ako CSI drivere)
```
AWS – EBS, EFS, FSx.
GCP – Persistent Disk, Filestore.
Azure – Disk, File.
OpenStack Cinder.
```

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
```
## Kubernetes StatefulSet Explained

A StatefulSet is designed for applications that require stable storage and network identities. This example demonstrates how it provides a unique identity and ties a persistent volume to each pod.

1. The Headless Service

The StatefulSet relies on a headless service to provide the stable network identity for each pod. A headless service has no ClusterIP and acts as a DNS record for the pods it controls.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  # A headless service has no ClusterIP, it only provides DNS records.
  # This is a requirement for StatefulSets.
  clusterIP: None 
  selector:
    app: nginx-statefulset
  ports:
  - port: 80
    name: web
  # The service must have the same name as the StatefulSet.
  # The domain name will be ..default.svc.cluster.local
  # Example: nginx-0.nginx-service.default.svc.cluster.local
  # This provides the stable, unique network identity for each pod.
```

2. The StatefulSet Manifest

This is the main manifest for our stateful application. It specifies the number of replicas, the pod template, and the volume claim template for persistent storage.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
spec:
  selector:
    matchLabels:
      app: nginx-statefulset
  serviceName: "nginx-service" # This must match the name of the headless service.
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx-statefulset
    spec:
      containers:
      - name: nginx-web
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www-data
          mountPath: /usr/share/nginx/html
  
  # This template creates a PersistentVolumeClaim for each pod.
  # The PVC will be named -
  # Example: www-data-nginx-statefulset-0
  volumeClaimTemplates:
  - metadata:
      name: www-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
