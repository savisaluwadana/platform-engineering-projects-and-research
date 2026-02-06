# Kubernetes Cheatsheet

## Quick Reference Guide for Platform Engineers

### kubectl Basics

#### Installation and Configuration
```bash
# Install kubectl (Linux)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Check version
kubectl version --client                     # Client version
kubectl version --short                      # Short version info

# Cluster information
kubectl cluster-info                         # Display cluster info
kubectl get nodes                            # List all nodes
kubectl get nodes -o wide                    # Detailed node info
kubectl describe node node-name              # Node details

# Context and configuration
kubectl config view                          # View kubeconfig
kubectl config get-contexts                  # List all contexts
kubectl config current-context               # Show current context
kubectl config use-context context-name      # Switch context
kubectl config set-context --current --namespace=my-namespace  # Set default namespace

# Cluster authentication
kubectl config set-credentials user --token=bearer_token
kubectl config set-cluster cluster-name --server=https://api-server:6443
kubectl config set-context context-name --cluster=cluster-name --user=user
```

#### kubectl Syntax and Common Flags
```bash
# Basic syntax
kubectl [command] [TYPE] [NAME] [flags]

# Common resource types (short names)
kubectl api-resources                        # List all resource types
# po = pods, svc = services, deploy = deployments
# rs = replicasets, ds = daemonsets, sts = statefulsets
# cm = configmaps, secret = secrets, ns = namespaces
# pv = persistentvolumes, pvc = persistentvolumeclaims
# ing = ingress, sa = serviceaccounts

# Common flags
-n, --namespace=NAMESPACE                    # Specify namespace
-A, --all-namespaces                         # Across all namespaces
-o, --output=FORMAT                          # Output format (yaml, json, wide, name)
-w, --watch                                  # Watch for changes
-l, --selector=SELECTOR                      # Filter by labels
--dry-run=client                             # Preview without creating
--force                                      # Force operation
-f, --filename=FILE                          # Specify manifest file
-k, --kustomize=DIR                          # Apply kustomize directory
--recursive                                  # Process directory recursively
```

### Pod Management

#### Creating and Running Pods
```bash
# Create pod from image
kubectl run nginx --image=nginx              # Simple pod
kubectl run nginx --image=nginx --port=80    # With container port
kubectl run nginx --image=nginx --dry-run=client -o yaml  # Generate YAML

# Run with specific settings
kubectl run busybox --image=busybox -it --rm -- /bin/sh  # Interactive, auto-delete
kubectl run nginx --image=nginx --env="ENV=prod"         # With env variable
kubectl run nginx --image=nginx --labels="app=web,env=prod"  # With labels
kubectl run nginx --image=nginx --requests='cpu=100m,memory=256Mi'  # Resource requests
kubectl run nginx --image=nginx --limits='cpu=200m,memory=512Mi'    # Resource limits

# Create from manifest
kubectl create -f pod.yaml                   # Create from file
kubectl apply -f pod.yaml                    # Apply/update from file
kubectl delete -f pod.yaml                   # Delete using file

# Create pod with command
kubectl run busybox --image=busybox -- echo "Hello World"
kubectl run busybox --image=busybox -- sleep 3600
```

#### Listing and Inspecting Pods
```bash
# List pods
kubectl get pods                             # Current namespace
kubectl get pods -n namespace-name           # Specific namespace
kubectl get pods -A                          # All namespaces
kubectl get pods -o wide                     # Show node, IP, etc.
kubectl get pods -o yaml                     # YAML output
kubectl get pods -o json                     # JSON output
kubectl get pods --show-labels               # Show labels
kubectl get pods -l app=nginx                # Filter by label
kubectl get pods --field-selector status.phase=Running  # Filter by field
kubectl get pods --sort-by=.metadata.creationTimestamp  # Sort by creation time

# Describe and details
kubectl describe pod pod-name                # Detailed information
kubectl get pod pod-name -o yaml            # Full YAML definition
kubectl get pod pod-name -o jsonpath='{.status.podIP}'  # Extract specific field

# Watch pods
kubectl get pods -w                          # Watch for changes
kubectl get pods -w -o wide                  # Watch with details
```

#### Pod Operations
```bash
# Execute commands in pod
kubectl exec pod-name -- ls /app             # Run command
kubectl exec -it pod-name -- /bin/bash       # Interactive shell
kubectl exec -it pod-name -c container-name -- /bin/sh  # Specific container

# View logs
kubectl logs pod-name                        # View logs
kubectl logs -f pod-name                     # Follow logs (tail -f)
kubectl logs pod-name -c container-name      # Specific container
kubectl logs --tail=100 pod-name            # Last 100 lines
kubectl logs --since=1h pod-name            # Last hour
kubectl logs --since-time=2024-01-01T00:00:00Z pod-name  # Since timestamp
kubectl logs -p pod-name                     # Previous container logs (after crash)
kubectl logs pod-name --all-containers       # All containers in pod

# Port forwarding
kubectl port-forward pod-name 8080:80        # Forward local 8080 to pod's 80
kubectl port-forward pod-name :80            # Random local port to pod's 80
kubectl port-forward pod-name 8080:80 --address=0.0.0.0  # Allow external access

# Copy files
kubectl cp file.txt pod-name:/path/file.txt  # Copy to pod
kubectl cp pod-name:/path/file.txt file.txt  # Copy from pod
kubectl cp pod-name:/path/file.txt file.txt -c container-name  # Specific container

# Delete pods
kubectl delete pod pod-name                  # Delete pod
kubectl delete pod pod-name --force --grace-period=0  # Force delete
kubectl delete pods --all                    # Delete all in namespace
kubectl delete pods -l app=nginx             # Delete by label
```

### Deployments

#### Creating Deployments
```bash
# Create deployment
kubectl create deployment nginx --image=nginx              # Simple deployment
kubectl create deployment nginx --image=nginx --replicas=3  # With replicas
kubectl create deployment nginx --image=nginx --port=80    # With container port
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml

# Apply deployment
kubectl apply -f deployment.yaml             # Create/update from file
kubectl apply -f deployments/ --recursive    # Apply all in directory

# Generate deployment YAML
kubectl create deployment nginx --image=nginx:1.21 --replicas=3 --dry-run=client -o yaml
```

#### Managing Deployments
```bash
# List deployments
kubectl get deployments                      # List deployments
kubectl get deploy -A                        # All namespaces
kubectl get deploy -o wide                   # More details
kubectl describe deployment nginx            # Detailed info

# Scale deployments
kubectl scale deployment nginx --replicas=5  # Scale to 5 replicas
kubectl scale deployment nginx --replicas=0  # Scale down to 0
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80  # Auto-scale

# Update deployments
kubectl set image deployment/nginx nginx=nginx:1.22  # Update image
kubectl set image deployment/nginx nginx=nginx:1.22 --record  # Record change
kubectl set env deployment/nginx ENV=production     # Set environment variable
kubectl set resources deployment nginx --limits=cpu=200m,memory=512Mi  # Update resources

# Rollout management
kubectl rollout status deployment/nginx      # Check rollout status
kubectl rollout history deployment/nginx     # View rollout history
kubectl rollout history deployment/nginx --revision=2  # Specific revision details
kubectl rollout undo deployment/nginx        # Rollback to previous version
kubectl rollout undo deployment/nginx --to-revision=2  # Rollback to specific revision
kubectl rollout restart deployment/nginx     # Restart deployment
kubectl rollout pause deployment/nginx       # Pause rollout
kubectl rollout resume deployment/nginx      # Resume rollout

# Edit deployment
kubectl edit deployment nginx                # Edit in default editor
kubectl patch deployment nginx -p '{"spec":{"replicas":5}}'  # Patch specific field

# Delete deployment
kubectl delete deployment nginx              # Delete deployment
```

### Services

#### Creating Services
```bash
# Create service (expose deployment/pod)
kubectl expose deployment nginx --port=80 --type=ClusterIP     # ClusterIP (default)
kubectl expose deployment nginx --port=80 --type=NodePort      # NodePort
kubectl expose deployment nginx --port=80 --type=LoadBalancer  # LoadBalancer
kubectl expose pod nginx --port=80 --name=nginx-service        # Expose pod

# Create service with specific settings
kubectl expose deployment nginx --port=80 --target-port=8080   # Different target port
kubectl expose deployment nginx --port=80 --name=nginx-svc     # Custom name

# Create service from YAML
kubectl create -f service.yaml
kubectl apply -f service.yaml

# Generate service YAML
kubectl create service clusterip nginx --tcp=80:80 --dry-run=client -o yaml
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
kubectl create service loadbalancer nginx --tcp=80:80 --dry-run=client -o yaml
```

#### Managing Services
```bash
# List services
kubectl get services                         # List services
kubectl get svc -A                          # All namespaces
kubectl get svc -o wide                     # More details
kubectl describe service nginx              # Detailed info

# Service endpoints
kubectl get endpoints                        # List all endpoints
kubectl get endpoints nginx                  # Specific service endpoints
kubectl describe endpoints nginx             # Endpoint details

# Port forwarding to service
kubectl port-forward service/nginx 8080:80   # Forward to service

# Delete service
kubectl delete service nginx                 # Delete service
kubectl delete svc --all                    # Delete all services
```

### ConfigMaps and Secrets

#### ConfigMaps
```bash
# Create ConfigMap from literal values
kubectl create configmap app-config --from-literal=key1=value1 --from-literal=key2=value2

# Create from file
kubectl create configmap app-config --from-file=config.properties
kubectl create configmap app-config --from-file=app-config=/path/to/file
kubectl create configmap app-config --from-file=configs/  # From directory

# Create from env file
kubectl create configmap app-config --from-env-file=config.env

# Create from YAML
kubectl apply -f configmap.yaml

# List and view
kubectl get configmaps                       # List ConfigMaps
kubectl get cm                              # Short form
kubectl describe configmap app-config       # Details
kubectl get configmap app-config -o yaml    # View as YAML

# Edit ConfigMap
kubectl edit configmap app-config            # Edit directly

# Delete ConfigMap
kubectl delete configmap app-config          # Delete
```

#### Secrets
```bash
# Create Secret from literal values
kubectl create secret generic db-secret --from-literal=username=admin --from-literal=password=secret123

# Create from file
kubectl create secret generic db-secret --from-file=username.txt --from-file=password.txt
kubectl create secret generic ssh-key --from-file=ssh-privatekey=/path/to/.ssh/id_rsa

# Create TLS secret
kubectl create secret tls tls-secret --cert=path/to/cert.crt --key=path/to/cert.key

# Create Docker registry secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=password \
  --docker-email=user@example.com

# Create from YAML (base64 encoded values)
kubectl apply -f secret.yaml

# List and view
kubectl get secrets                          # List secrets
kubectl describe secret db-secret           # Details (values hidden)
kubectl get secret db-secret -o yaml        # View as YAML (base64 encoded)
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 --decode  # Decode value

# Edit secret
kubectl edit secret db-secret                # Edit directly

# Delete secret
kubectl delete secret db-secret              # Delete
```

### Namespace Management

```bash
# List namespaces
kubectl get namespaces                       # List all namespaces
kubectl get ns                              # Short form

# Create namespace
kubectl create namespace dev                 # Create namespace
kubectl create ns prod                       # Short form

# Create from YAML
kubectl apply -f namespace.yaml

# Describe namespace
kubectl describe namespace dev               # Namespace details

# Set default namespace for context
kubectl config set-context --current --namespace=dev

# View resources in namespace
kubectl get all -n dev                       # All resources
kubectl get pods -n dev                      # Pods in namespace

# Delete namespace (deletes all resources in it)
kubectl delete namespace dev                 # Delete namespace
kubectl delete ns dev --force --grace-period=0  # Force delete

# Resource quotas in namespace
kubectl get resourcequota -n dev             # View quotas
kubectl describe resourcequota -n dev        # Quota details

# Limit ranges in namespace
kubectl get limitrange -n dev                # View limits
kubectl describe limitrange -n dev           # Limit details
```

### Resource Management and Scaling

#### Resource Requests and Limits
```bash
# Set resource requests and limits
kubectl set resources deployment nginx --requests=cpu=100m,memory=256Mi
kubectl set resources deployment nginx --limits=cpu=500m,memory=512Mi
kubectl set resources deployment nginx --requests=cpu=100m,memory=256Mi --limits=cpu=500m,memory=512Mi

# View resource usage
kubectl top nodes                            # Node resource usage
kubectl top pods                            # Pod resource usage
kubectl top pods -A                         # All namespaces
kubectl top pods -n namespace --sort-by=memory  # Sort by memory
kubectl top pods -n namespace --sort-by=cpu     # Sort by CPU

# Resource quotas
kubectl create quota dev-quota --hard=cpu=2,memory=4Gi,pods=10 -n dev
kubectl get resourcequota -n dev
kubectl describe resourcequota dev-quota -n dev

# Limit ranges
kubectl create -f limitrange.yaml
kubectl get limitrange -n dev
kubectl describe limitrange -n dev
```

#### Horizontal Pod Autoscaler (HPA)
```bash
# Create HPA
kubectl autoscale deployment nginx --cpu-percent=80 --min=2 --max=10
kubectl autoscale deployment nginx --cpu-percent=70 --min=3 --max=15 --name=nginx-hpa

# Create from YAML
kubectl apply -f hpa.yaml

# List and view HPA
kubectl get hpa                              # List HPAs
kubectl get hpa -A                          # All namespaces
kubectl describe hpa nginx-hpa              # HPA details

# Edit HPA
kubectl edit hpa nginx-hpa                   # Edit HPA

# Delete HPA
kubectl delete hpa nginx-hpa                 # Delete HPA
```

#### Vertical Pod Autoscaler (VPA)
```bash
# VPA requires VPA controller to be installed
# Create VPA
kubectl apply -f vpa.yaml

# List VPA
kubectl get vpa                              # List VPAs
kubectl describe vpa nginx-vpa              # VPA details

# Delete VPA
kubectl delete vpa nginx-vpa                 # Delete VPA
```

### Networking

#### Services Types
```bash
# ClusterIP (internal only)
kubectl create service clusterip nginx --tcp=80:80

# NodePort (external access via node IP:port)
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080

# LoadBalancer (cloud load balancer)
kubectl create service loadbalancer nginx --tcp=80:80

# ExternalName (DNS CNAME)
kubectl create service externalname ext-svc --external-name=external.example.com

# Headless service (no ClusterIP for StatefulSets)
# Set clusterIP: None in service YAML
```

#### Ingress
```bash
# List ingress
kubectl get ingress                          # List ingress resources
kubectl get ing -A                          # All namespaces
kubectl describe ingress my-ingress         # Ingress details

# Create ingress
kubectl apply -f ingress.yaml

# Create simple ingress
kubectl create ingress simple --rule="example.com/=nginx:80" --dry-run=client -o yaml

# Create ingress with TLS
kubectl create ingress tls-ingress \
  --rule="example.com/=nginx:80,tls=tls-secret" \
  --dry-run=client -o yaml

# Edit ingress
kubectl edit ingress my-ingress              # Edit ingress

# Delete ingress
kubectl delete ingress my-ingress            # Delete ingress

# Ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

#### Network Policies
```bash
# List network policies
kubectl get networkpolicy                    # List policies
kubectl get netpol -A                       # All namespaces
kubectl describe networkpolicy my-policy    # Policy details

# Create network policy
kubectl apply -f networkpolicy.yaml

# Example: Deny all ingress traffic
kubectl apply -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF

# Delete network policy
kubectl delete networkpolicy my-policy       # Delete policy
```

#### DNS and Service Discovery
```bash
# Service DNS format
# <service-name>.<namespace>.svc.cluster.local

# Test DNS from pod
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup nginx
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup nginx.default.svc.cluster.local

# View CoreDNS logs
kubectl logs -n kube-system -l k8s-app=kube-dns

# CoreDNS ConfigMap
kubectl get configmap coredns -n kube-system -o yaml
```

### Storage

#### Persistent Volumes (PV)
```bash
# List PVs
kubectl get pv                               # List all PVs
kubectl get pv -o wide                      # More details
kubectl describe pv pv-name                 # PV details

# Create PV
kubectl apply -f pv.yaml

# Delete PV
kubectl delete pv pv-name                    # Delete PV

# PV status
# Available - ready to be bound
# Bound - bound to PVC
# Released - PVC deleted but resource not reclaimed
# Failed - automatic reclamation failed
```

#### Persistent Volume Claims (PVC)
```bash
# List PVCs
kubectl get pvc                              # List PVCs
kubectl get pvc -A                          # All namespaces
kubectl describe pvc pvc-name               # PVC details

# Create PVC
kubectl apply -f pvc.yaml

# Create simple PVC
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
EOF

# Delete PVC
kubectl delete pvc pvc-name                  # Delete PVC

# View PVC in use
kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.volumes[*].persistentVolumeClaim.claimName}{"\n"}{end}'
```

#### Storage Classes
```bash
# List storage classes
kubectl get storageclass                     # List storage classes
kubectl get sc                              # Short form
kubectl describe storageclass standard      # SC details

# Create storage class
kubectl apply -f storageclass.yaml

# Set default storage class
kubectl patch storageclass standard -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

# Remove default annotation
kubectl patch storageclass standard -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'

# Delete storage class
kubectl delete storageclass my-sc            # Delete storage class
```

#### Volume Operations
```bash
# List volumes in pods
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.volumes[*].name}{"\n"}{end}'

# Describe volume mounts
kubectl describe pod pod-name | grep -A 5 Mounts

# Check volume usage in pod
kubectl exec pod-name -- df -h
kubectl exec pod-name -- du -sh /data
```

### RBAC and Security

#### Service Accounts
```bash
# List service accounts
kubectl get serviceaccounts                  # List SAs
kubectl get sa                              # Short form
kubectl describe sa my-sa                   # SA details

# Create service account
kubectl create serviceaccount my-sa          # Create SA
kubectl create sa build-bot -n ci-cd        # In specific namespace

# Create from YAML
kubectl apply -f serviceaccount.yaml

# View SA token
kubectl get secret $(kubectl get sa my-sa -o jsonpath='{.secrets[0].name}') -o yaml

# Delete service account
kubectl delete serviceaccount my-sa          # Delete SA
```

#### Roles and RoleBindings (namespace-scoped)
```bash
# List roles
kubectl get roles                            # List roles
kubectl get roles -A                        # All namespaces
kubectl describe role my-role               # Role details

# Create role
kubectl create role pod-reader --verb=get,list,watch --resource=pods
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n dev

# Create from YAML
kubectl apply -f role.yaml

# List role bindings
kubectl get rolebindings                     # List role bindings
kubectl get rolebindings -A                 # All namespaces
kubectl describe rolebinding my-binding     # Binding details

# Create role binding
kubectl create rolebinding read-pods --role=pod-reader --user=jane
kubectl create rolebinding read-pods --role=pod-reader --serviceaccount=default:my-sa
kubectl create rolebinding read-pods --role=pod-reader --group=developers

# Delete role and bindings
kubectl delete role my-role                  # Delete role
kubectl delete rolebinding my-binding        # Delete binding
```

#### ClusterRoles and ClusterRoleBindings (cluster-scoped)
```bash
# List cluster roles
kubectl get clusterroles                     # List cluster roles
kubectl describe clusterrole view           # ClusterRole details

# Create cluster role
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods
kubectl create clusterrole node-reader --verb=get,list --resource=nodes

# Create from YAML
kubectl apply -f clusterrole.yaml

# List cluster role bindings
kubectl get clusterrolebindings              # List cluster role bindings
kubectl describe clusterrolebinding my-binding  # Binding details

# Create cluster role binding
kubectl create clusterrolebinding read-pods --clusterrole=pod-reader --user=jane
kubectl create clusterrolebinding read-pods --clusterrole=pod-reader --serviceaccount=default:my-sa
kubectl create clusterrolebinding read-nodes --clusterrole=node-reader --group=ops-team

# Delete cluster role and bindings
kubectl delete clusterrole my-role           # Delete cluster role
kubectl delete clusterrolebinding my-binding # Delete binding
```

#### Authorization Checks
```bash
# Check if you can perform action
kubectl auth can-i create pods               # Can I create pods?
kubectl auth can-i create pods -n dev       # In specific namespace
kubectl auth can-i '*' '*'                  # Am I cluster admin?

# Check for other users
kubectl auth can-i get pods --as=jane        # Can jane get pods?
kubectl auth can-i create deployments --as=system:serviceaccount:default:my-sa  # Can SA create deployments?

# Reconcile RBAC
kubectl auth reconcile -f rbac.yaml          # Reconcile RBAC resources
```

#### Security Context
```bash
# View pod security context
kubectl get pod pod-name -o jsonpath='{.spec.securityContext}'

# View container security context
kubectl get pod pod-name -o jsonpath='{.spec.containers[0].securityContext}'

# Run pod with security context
kubectl run nginx --image=nginx --dry-run=client -o yaml | \
  kubectl patch --dry-run=client -o yaml -f - -p '
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: nginx
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
      readOnlyRootFilesystem: true
'
```

#### Pod Security Standards/Policies
```bash
# Label namespace with pod security standard
kubectl label namespace default pod-security.kubernetes.io/enforce=baseline
kubectl label namespace default pod-security.kubernetes.io/warn=restricted
kubectl label namespace default pod-security.kubernetes.io/audit=restricted

# View pod security labels
kubectl get namespace default --show-labels

# Pod security levels: privileged, baseline, restricted
```

### Troubleshooting

#### Debugging Pods
```bash
# Check pod status
kubectl get pods                             # Basic status
kubectl get pods -o wide                    # With node and IP
kubectl describe pod pod-name               # Detailed status and events

# Common pod states
# Pending - waiting to be scheduled
# ContainerCreating - pulling image/mounting volumes
# Running - all containers running
# CrashLoopBackOff - container repeatedly crashing
# ImagePullBackOff - can't pull image
# Error - container exited with error
# Completed - container exited successfully

# View logs
kubectl logs pod-name                        # Current logs
kubectl logs pod-name --previous            # Previous container logs (after crash)
kubectl logs pod-name -c container-name     # Specific container
kubectl logs -f pod-name                    # Follow logs
kubectl logs --tail=100 pod-name            # Last 100 lines
kubectl logs --since=1h pod-name            # Last hour

# Execute commands in pod
kubectl exec -it pod-name -- /bin/bash       # Interactive shell
kubectl exec pod-name -- env                # View environment variables
kubectl exec pod-name -- cat /etc/resolv.conf  # Check DNS config
kubectl exec pod-name -- nslookup service-name # Test DNS resolution
kubectl exec pod-name -- curl http://service-name  # Test service connectivity

# Debug with ephemeral containers (k8s 1.23+)
kubectl debug pod-name -it --image=busybox --target=container-name
kubectl debug pod-name -it --image=nicolaka/netshoot --share-processes

# Copy pod for debugging
kubectl debug pod-name -it --copy-to=pod-name-debug --container=container-name -- sh

# Debug node
kubectl debug node/node-name -it --image=ubuntu
```

#### Debugging Services
```bash
# Check service
kubectl get svc service-name                 # Service info
kubectl describe svc service-name           # Detailed info
kubectl get endpoints service-name          # Service endpoints

# No endpoints? Check:
# 1. Pod labels match service selector
kubectl get pods --show-labels
kubectl get svc service-name -o yaml | grep selector -A 5

# 2. Pods are running
kubectl get pods -l app=myapp

# Test service connectivity
kubectl run test-pod --image=busybox -it --rm -- wget -O- http://service-name:port
kubectl run test-pod --image=nicolaka/netshoot -it --rm -- curl http://service-name:port

# Test from inside pod
kubectl exec -it pod-name -- curl http://service-name:port
kubectl exec -it pod-name -- nslookup service-name
```

#### Debugging Network
```bash
# Test DNS
kubectl run test-dns --image=busybox -it --rm -- nslookup kubernetes.default
kubectl exec -it pod-name -- nslookup service-name.namespace.svc.cluster.local

# Test connectivity
kubectl run test-net --image=nicolaka/netshoot -it --rm -- bash
# Inside container: curl, ping, traceroute, nslookup, dig, etc.

# Check network policies
kubectl get networkpolicy -A                 # List all policies
kubectl describe networkpolicy policy-name  # Policy details

# Check CNI plugin logs
kubectl logs -n kube-system -l k8s-app=calico-node  # Calico
kubectl logs -n kube-system -l k8s-app=flannel      # Flannel
kubectl logs -n kube-system -l app=weave-net        # Weave

# Test pod-to-pod connectivity
kubectl exec pod1 -- ping -c 3 <pod2-ip>
```

#### Debugging Storage
```bash
# Check PV and PVC
kubectl get pv                               # PersistentVolumes
kubectl get pvc                             # PersistentVolumeClaims
kubectl describe pv pv-name                 # PV details
kubectl describe pvc pvc-name               # PVC details

# PVC stuck in Pending?
# Check: storage class exists, enough capacity, access modes match

# Check storage class
kubectl get storageclass                     # Available storage classes
kubectl describe storageclass sc-name       # SC details

# Check volume mounts in pod
kubectl describe pod pod-name | grep -A 10 Volumes
kubectl describe pod pod-name | grep -A 5 Mounts

# Check disk usage
kubectl exec pod-name -- df -h
kubectl exec pod-name -- du -sh /mount-path
```

#### Cluster Troubleshooting
```bash
# Check cluster health
kubectl get --raw /healthz                   # API server health
kubectl get componentstatuses                # Component status (deprecated)
kubectl get nodes                           # Node status

# Check node conditions
kubectl describe node node-name | grep -A 10 Conditions
# Ready, MemoryPressure, DiskPressure, PIDPressure, NetworkUnavailable

# Check system pods
kubectl get pods -n kube-system             # System pods
kubectl get pods -n kube-system -o wide     # With node info

# Check events
kubectl get events                          # Namespace events
kubectl get events -A                       # All namespaces
kubectl get events --sort-by='.lastTimestamp'  # Recent events
kubectl get events --field-selector type=Warning  # Warning events only

# API server logs
kubectl logs -n kube-system kube-apiserver-node-name

# Controller manager logs
kubectl logs -n kube-system kube-controller-manager-node-name

# Scheduler logs
kubectl logs -n kube-system kube-scheduler-node-name

# etcd logs
kubectl logs -n kube-system etcd-node-name
```

#### Common Issues and Solutions
```bash
# ImagePullBackOff
# Check: image name, tag, registry credentials, network connectivity
kubectl describe pod pod-name | grep -A 10 Events
kubectl get secret regcred -o yaml          # Check registry secret

# CrashLoopBackOff
# Check: logs, liveness probe, resource limits, dependencies
kubectl logs pod-name --previous            # Previous container logs
kubectl describe pod pod-name              # Check restart count and events

# Pending pods
# Check: resource availability, node selectors, taints/tolerations, PVC binding
kubectl describe pod pod-name              # Check events
kubectl get nodes -o wide                  # Check node resources
kubectl top nodes                          # Resource usage

# Pod eviction
# Check: resource pressure, priority class
kubectl describe node node-name | grep -A 10 Conditions
kubectl get pods --field-selector status.phase=Failed

# Service not accessible
# Check: endpoints, labels, network policies, port configuration
kubectl get endpoints service-name
kubectl describe networkpolicy -A
```

### Helm Basics

#### Helm Installation
```bash
# Install Helm (Linux)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Check version
helm version                                 # Helm version
```

#### Helm Repository Management
```bash
# Add repository
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami

# List repositories
helm repo list                               # List all repos

# Update repositories
helm repo update                             # Update all repos

# Search charts
helm search repo nginx                       # Search in all repos
helm search repo bitnami/nginx              # Search in specific repo
helm search hub wordpress                    # Search Helm Hub

# Remove repository
helm repo remove stable                      # Remove repo
```

#### Installing Charts
```bash
# Install chart
helm install my-release bitnami/nginx       # Install chart
helm install my-release bitnami/nginx -n dev  # In specific namespace
helm install my-release bitnami/nginx --create-namespace -n dev  # Create namespace

# Install with custom values
helm install my-release bitnami/nginx --set service.type=LoadBalancer
helm install my-release bitnami/nginx --set replicaCount=3,service.port=8080
helm install my-release bitnami/nginx -f values.yaml  # From file
helm install my-release bitnami/nginx -f values1.yaml -f values2.yaml  # Multiple files

# Generate name automatically
helm install bitnami/nginx --generate-name   # Auto-generate release name

# Dry run
helm install my-release bitnami/nginx --dry-run --debug  # Test installation

# Install specific version
helm install my-release bitnami/nginx --version 13.2.23

# Install from local chart
helm install my-release ./mychart            # Local directory
helm install my-release mychart-0.1.0.tgz   # Local archive
```

#### Managing Releases
```bash
# List releases
helm list                                    # Current namespace
helm list -A                                # All namespaces
helm list -n dev                            # Specific namespace
helm list --all-namespaces                  # All namespaces (verbose)

# Get release info
helm status my-release                       # Release status
helm get all my-release                     # All release info
helm get values my-release                  # Values used
helm get manifest my-release                # Kubernetes manifests
helm get notes my-release                   # Release notes

# Upgrade release
helm upgrade my-release bitnami/nginx        # Upgrade to latest
helm upgrade my-release bitnami/nginx --version 13.2.24  # Specific version
helm upgrade my-release bitnami/nginx --set replicaCount=5  # With new values
helm upgrade my-release bitnami/nginx -f values.yaml  # From file
helm upgrade my-release bitnami/nginx --reuse-values  # Keep existing values
helm upgrade --install my-release bitnami/nginx  # Install if not exists

# Rollback release
helm rollback my-release                     # Rollback to previous
helm rollback my-release 2                  # Rollback to revision 2

# Release history
helm history my-release                      # Release history

# Uninstall release
helm uninstall my-release                    # Uninstall release
helm uninstall my-release -n dev            # From specific namespace
helm uninstall my-release --keep-history    # Keep history
```

#### Creating Charts
```bash
# Create new chart
helm create mychart                          # Create chart skeleton

# Validate chart
helm lint mychart                            # Check for issues

# Package chart
helm package mychart                         # Create .tgz archive

# Show chart info
helm show chart bitnami/nginx               # Chart metadata
helm show values bitnami/nginx              # Default values
helm show readme bitnami/nginx              # README
helm show all bitnami/nginx                 # All info

# Template rendering
helm template my-release bitnami/nginx       # Render templates
helm template my-release bitnami/nginx -f values.yaml  # With custom values
helm template my-release ./mychart --debug   # Debug mode

# Dependency management
helm dependency list mychart                 # List dependencies
helm dependency update mychart               # Update dependencies
```

#### Helm Plugins
```bash
# List plugins
helm plugin list                             # List installed plugins

# Install plugin
helm plugin install https://github.com/databus23/helm-diff

# Useful plugins:
# helm-diff - show diff between releases
# helm-secrets - manage secrets
# helm-git - install from git repos
```

### Common Patterns and Best Practices

#### Labels and Annotations
```bash
# Common labels
app: myapp                                   # Application name
version: v1.2.3                             # Application version
component: frontend                          # Component name
environment: production                      # Environment
tier: frontend                              # Application tier
managed-by: helm                            # Tool managing resource

# Kubernetes recommended labels
app.kubernetes.io/name: myapp
app.kubernetes.io/instance: myapp-prod
app.kubernetes.io/version: "1.2.3"
app.kubernetes.io/component: frontend
app.kubernetes.io/part-of: myplatform
app.kubernetes.io/managed-by: helm

# Common annotations
kubernetes.io/change-cause: "Update to v1.2.3"  # Rollout change cause
prometheus.io/scrape: "true"                     # Prometheus scraping
prometheus.io/port: "9090"                       # Prometheus port

# Apply labels
kubectl label pods pod-name env=prod         # Add label
kubectl label pods pod-name env=dev --overwrite  # Update label
kubectl label pods pod-name env-             # Remove label
kubectl label pods -l app=nginx tier=frontend  # Label multiple pods
```

#### Health Checks
```yaml
# Liveness probe (restart if unhealthy)
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

# Readiness probe (remove from service if not ready)
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  successThreshold: 1

# Startup probe (for slow-starting containers)
startupProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

#### Resource Patterns
```yaml
# Resource requests and limits
resources:
  requests:
    memory: "256Mi"
    cpu: "100m"
  limits:
    memory: "512Mi"
    cpu: "500m"

# Common CPU values
# 100m = 0.1 CPU (10% of one core)
# 250m = 0.25 CPU
# 500m = 0.5 CPU
# 1 or 1000m = 1 full CPU core

# Common memory values
# 128Mi = 128 mebibytes
# 256Mi = 256 mebibytes
# 512Mi = 512 mebibytes
# 1Gi = 1 gibibyte
```

#### Multi-Container Patterns
```bash
# Sidecar pattern - helper container
# Example: logging agent, proxy

# Ambassador pattern - proxy for external services
# Example: connection pooling, retries

# Adapter pattern - standardize output
# Example: log format conversion

# Init containers - setup before main container
# Example: database migrations, config generation
```

#### ConfigMap and Secret Usage
```yaml
# Environment variables from ConfigMap
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: log_level

# Environment variables from Secret
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password

# All keys as environment variables
envFrom:
  - configMapRef:
      name: app-config
  - secretRef:
      name: app-secrets

# Volume mount from ConfigMap
volumes:
  - name: config
    configMap:
      name: app-config
volumeMounts:
  - name: config
    mountPath: /etc/config

# Volume mount from Secret
volumes:
  - name: secrets
    secret:
      secretName: app-secrets
volumeMounts:
  - name: secrets
    mountPath: /etc/secrets
    readOnly: true
```

### Production Deployment Tips

#### Deployment Strategies
```yaml
# Rolling update (default)
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1        # Max pods above desired count
    maxUnavailable: 0  # Max pods unavailable during update

# Recreate (downtime acceptable)
strategy:
  type: Recreate

# Blue-Green deployment (use labels and services)
# Canary deployment (use multiple deployments with different weights)
```

#### High Availability
```bash
# Multiple replicas
kubectl scale deployment myapp --replicas=3

# Pod disruption budget
kubectl create pdb myapp-pdb --selector=app=myapp --min-available=2
kubectl create pdb myapp-pdb --selector=app=myapp --max-unavailable=1

# Anti-affinity (spread across nodes)
# Use pod anti-affinity in deployment spec

# Multiple availability zones
# Use node labels and node affinity
```

#### Monitoring and Logging
```bash
# Resource monitoring
kubectl top nodes
kubectl top pods -A

# Events
kubectl get events --sort-by='.lastTimestamp'

# Logs
kubectl logs -f deployment/myapp
kubectl logs -f -l app=myapp --all-containers=true

# Prometheus annotations
prometheus.io/scrape: "true"
prometheus.io/port: "9090"
prometheus.io/path: "/metrics"
```

#### Security Best Practices
```yaml
# Non-root user
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000

# Read-only root filesystem
securityContext:
  readOnlyRootFilesystem: true

# Drop capabilities
securityContext:
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE

# No privilege escalation
securityContext:
  allowPrivilegeEscalation: false

# Resource limits
resources:
  limits:
    memory: "512Mi"
    cpu: "500m"
  requests:
    memory: "256Mi"
    cpu: "100m"

# Network policies (default deny)
# Use RBAC with least privilege
# Encrypt secrets at rest
# Use private registries with image scanning
```

#### Backup and Disaster Recovery
```bash
# Backup etcd (critical!)
ETCDCTL_API=3 etcdctl snapshot save snapshot.db

# Backup resources
kubectl get all --all-namespaces -o yaml > backup.yaml

# Backup specific resources
kubectl get deployment,svc,configmap,secret -n prod -o yaml > prod-backup.yaml

# Use Velero for comprehensive backups
# Regular backups of persistent volumes
# Test restore procedures regularly
```

#### Performance Optimization
```bash
# Use resource requests and limits
# Enable horizontal pod autoscaling
# Use persistent volume for stateful apps
# Implement proper health checks
# Use readiness probes to control traffic
# Configure appropriate timeout values
# Use node affinity for optimal placement
# Implement pod priority classes
# Use local storage for cache when possible
# Monitor and tune resource usage

# Pod priority
kubectl create priorityclass high-priority --value=1000 --description="High priority"
kubectl create priorityclass low-priority --value=100 --description="Low priority"
```

#### Update and Maintenance
```bash
# Drain node for maintenance
kubectl drain node-name --ignore-daemonsets --delete-emptydir-data

# Cordon node (mark unschedulable)
kubectl cordon node-name

# Uncordon node (mark schedulable)
kubectl uncordon node-name

# Update deployments with rollout control
kubectl set image deployment/myapp myapp=myapp:v2 --record
kubectl rollout status deployment/myapp
kubectl rollout pause deployment/myapp    # Pause if issues
kubectl rollout undo deployment/myapp     # Rollback if needed

# Regular updates
# Update Kubernetes cluster regularly
# Keep container images updated
# Update Helm charts and dependencies
# Review and update RBAC policies
```

#### Cost Optimization
```bash
# Right-size resource requests
# Use cluster autoscaler
# Implement pod disruption budgets
# Use spot/preemptible instances for non-critical workloads
# Clean up unused resources
kubectl delete pods --field-selector status.phase=Failed
kubectl delete pods --field-selector status.phase=Succeeded

# Use namespace quotas
# Monitor resource usage with top
kubectl top nodes
kubectl top pods -A --sort-by=memory
kubectl top pods -A --sort-by=cpu

# Identify unused PVCs
# Implement pod priority for cost-effective scheduling
```

## Quick Tips

```bash
# Use aliases for efficiency
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployments'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias kx='kubectl exec -it'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'

# Enable kubectl autocomplete
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
complete -F __start_kubectl k  # If using alias 'k'

# Use short names
kubectl get po         # pods
kubectl get svc        # services
kubectl get deploy     # deployments
kubectl get rs         # replicasets
kubectl get cm         # configmaps
kubectl get ns         # namespaces
kubectl get no         # nodes

# Use -o wide for more info
kubectl get pods -o wide
kubectl get nodes -o wide

# Use --dry-run for generating YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml
kubectl expose deployment nginx --port=80 --dry-run=client -o yaml

# Use explain for field documentation
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers

# Watch resources
kubectl get pods -w
kubectl get events -w

# Multiple resources at once
kubectl get pods,svc,deploy

# Context and namespace shortcuts
kubectl config set-context --current --namespace=dev
kubectl get pods --all-namespaces
kubectl get pods -A

# JSONPath for custom output
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.podIP}{"\n"}{end}'

# Force delete stuck resources
kubectl delete pod pod-name --force --grace-period=0
kubectl patch pod pod-name -p '{"metadata":{"finalizers":null}}'
```

---

**Note**: This cheatsheet covers Kubernetes fundamentals and common operations. Always refer to official documentation for the latest features and best practices. Test commands in a non-production environment first.

**Official Documentation**: https://kubernetes.io/docs/
