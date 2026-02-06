# Container Orchestration with Kubernetes

## Theory

### Core Concepts

#### Kubernetes Architecture: Orchestrating Containers at Scale
Kubernetes (K8s) is a production-grade container orchestration platform with a distributed architecture:

**Control Plane Components** (the "brain" of the cluster):
- **API Server (kube-apiserver)**: Front-end for the Kubernetes control plane, exposing the Kubernetes API. All communication with the cluster goes through this component. Validates and processes REST requests.
- **etcd**: Distributed key-value store serving as Kubernetes' backing store for all cluster data. Provides strong consistency and high availability. Critical for cluster state - always backup etcd!
- **Scheduler (kube-scheduler)**: Watches for newly created pods with no assigned node and selects an optimal node based on resource requirements, constraints, affinity rules, and data locality.
- **Controller Manager (kube-controller-manager)**: Runs controller processes that regulate cluster state (Node Controller, Replication Controller, Endpoints Controller, Service Account Controller).
- **Cloud Controller Manager**: Integrates with cloud provider APIs for node management, load balancers, and storage volumes.

**Worker Node Components** (where workloads run):
- **kubelet**: Primary node agent ensuring containers run in pods. Communicates with control plane and manages pod lifecycle.
- **kube-proxy**: Network proxy maintaining network rules for pod communication. Implements part of the Kubernetes Service concept.
- **Container Runtime**: Software running containers (containerd, CRI-O, Docker). Must implement the Kubernetes Container Runtime Interface (CRI).

Understanding this architecture helps diagnose issues, plan capacity, and optimize cluster performance.

#### Core Resources: Building Blocks of Kubernetes
Kubernetes uses declarative configuration to manage these fundamental resources:

- **Pods**: Smallest deployable units, containing one or more containers sharing network and storage. Pods are ephemeral - designed to be replaced, not repaired.
- **Deployments**: Declaratively manage pod replicas with rolling updates and rollbacks. Define desired state; controllers ensure actual state matches. Support strategies like RollingUpdate and Recreate.
- **ReplicaSets**: Ensure a specified number of pod replicas run at all times. Usually managed by Deployments rather than created directly.
- **Services**: Stable network endpoints for accessing pods. Abstract pod IP addresses and provide load balancing. Types include ClusterIP (internal), NodePort (external via node), LoadBalancer (cloud provider integration), and ExternalName (DNS CNAME).
- **ConfigMaps**: Store non-confidential configuration data as key-value pairs. Decouple configuration from container images for portability.
- **Secrets**: Store sensitive information (passwords, tokens, keys) with base64 encoding. Can be mounted as volumes or exposed as environment variables. Consider external secret management for production.

These resources enable deploying and managing containerized applications declaratively.

#### Kubernetes Networking: Connecting the Pieces
Kubernetes networking is complex but powerful, built on several key principles:

**Network Model**:
- Every pod gets its own IP address
- Pods can communicate with all other pods without NAT
- Nodes can communicate with all pods without NAT
- Container Network Interface (CNI) plugins implement this model (Calico, Flannel, Weave, Cilium)

**Service Types**:
- **ClusterIP**: Default type, provides internal cluster access only. Virtual IP managed by kube-proxy.
- **NodePort**: Exposes service on each node's IP at a static port (30000-32767). Routes to ClusterIP service.
- **LoadBalancer**: Creates external load balancer in cloud environments. Routes to NodePort service.
- **ExternalName**: Maps service to external DNS name for accessing external services.

**Ingress**: 
- Layer 7 (HTTP/HTTPS) load balancing and routing
- Provides hostname and path-based routing to services
- SSL/TLS termination
- Requires Ingress Controller (Nginx, Traefik, HAProxy, Istio Gateway)
- Supports virtual hosting and path-based routing

**Network Policies**: Firewall rules controlling pod-to-pod communication using labels. Requires CNI plugin support (Calico, Cilium, Weave).

Mastering Kubernetes networking enables building secure, scalable microservices architectures.

#### Storage in Kubernetes: Persisting Data
Kubernetes provides abstraction layers for storage management:

- **Volumes**: Directory accessible to containers in a pod. Types include emptyDir (ephemeral), hostPath (node filesystem), configMap, secret, and cloud provider volumes (AWS EBS, Azure Disk, GCP Persistent Disk).
- **PersistentVolumes (PV)**: Cluster-wide storage resources provisioned by administrators or dynamically via StorageClasses. Lifecycle independent of pods.
- **PersistentVolumeClaims (PVC)**: Requests for storage by users. Pods reference PVCs to mount persistent storage.
- **StorageClasses**: Define different storage tiers (SSD, HDD, regional, zonal) with different performance characteristics and cost. Enable dynamic provisioning.
- **Volume Modes**: Filesystem (mounted as directory) or Block (raw block device).
- **Access Modes**: ReadWriteOnce (single node), ReadOnlyMany (multiple nodes read-only), ReadWriteMany (multiple nodes read-write).

Dynamic provisioning with StorageClasses simplifies storage management and enables self-service for development teams.

#### Configuration Management: Separating Config from Code
Kubernetes promotes the 12-factor app principle of externalizing configuration:

- **ConfigMaps**: Store configuration files, command-line arguments, environment variables. Can be consumed as:
  - Environment variables
  - Command-line arguments
  - Files in a volume
- **Secrets**: Similar to ConfigMaps but designed for sensitive data. Stored base64-encoded (NOT encrypted by default). Mount as files or environment variables.
- **Environment Variables**: Can reference ConfigMap or Secret values directly.
- **External Secret Management**: Integrate with HashiCorp Vault, AWS Secrets Manager, Azure Key Vault for better security and rotation.

Proper configuration management enables environment promotion (dev → staging → prod) without rebuilding images.

#### Scaling: Meeting Demand Automatically
Kubernetes provides multiple scaling mechanisms:

- **Horizontal Pod Autoscaler (HPA)**: Automatically scales replica count based on metrics (CPU, memory, custom metrics). Queries Metrics Server for resource utilization.
- **Vertical Pod Autoscaler (VPA)**: Adjusts CPU and memory requests/limits for containers based on usage patterns. Helps right-size resources.
- **Cluster Autoscaler**: Adds or removes nodes based on pod scheduling needs. Integrates with cloud providers.
- **Manual Scaling**: Use `kubectl scale` to manually adjust replicas.
- **Metrics**: HPA uses Metrics Server for resource metrics and Custom Metrics API for application-specific metrics (from Prometheus, etc.).

Auto-scaling ensures efficient resource utilization while maintaining performance during traffic spikes.

#### Security: Defense in Depth
Kubernetes security requires multiple layers:

- **RBAC (Role-Based Access Control)**: Controls access to Kubernetes API. Define Roles (namespace-scoped) or ClusterRoles (cluster-wide) with permissions, bind to users/groups with RoleBindings or ClusterRoleBindings. Follow least privilege principle.
- **Network Policies**: Control pod-to-pod communication at the IP and port level. Define ingress and egress rules using pod selectors and namespaces.
- **Pod Security Standards**: Enforce security policies (Privileged, Baseline, Restricted) using Pod Security admission controller. Replaces deprecated PodSecurityPolicy.
- **Service Mesh**: Adds encryption (mTLS), authentication, and authorization to service-to-service communication. Options include Istio, Linkerd, Consul.
- **Secrets Management**: Use external secret managers (Vault, CSI Secrets Driver) rather than native Secrets for production.
- **Image Security**: Scan images for vulnerabilities, use private registries, enforce image signing (admission webhooks).
- **Runtime Security**: Monitor container behavior with tools like Falco for detecting anomalies.

Layered security practices protect against breaches and ensure compliance.

#### Advanced Workload Types: Beyond Simple Deployments
Kubernetes supports various workload patterns:

- **StatefulSets**: For stateful applications requiring stable network identities, persistent storage, and ordered deployment/scaling (databases, distributed systems). Each pod gets a predictable name and PVC.
- **DaemonSets**: Ensure a pod runs on all (or selected) nodes. Used for node-level operations like log collection, monitoring agents, and network plugins.
- **Jobs**: Run pods to completion for batch processing. Ensures specified number of successful completions.
- **CronJobs**: Schedule jobs at specific times or intervals using cron syntax. Useful for backups, reports, and periodic maintenance.
- **Custom Resources (CRDs)**: Extend Kubernetes API with custom resource types. Combined with controllers/operators, enable managing complex applications declaratively.
- **Operators**: Kubernetes-native applications packaging, deploying, and managing other applications using CRDs and custom controllers. Encode operational knowledge into software.

Understanding these workload types enables running diverse applications on Kubernetes.

### Key Skills
1. **Deploy and manage applications on Kubernetes**: Write YAML manifests for deployments, services, and ingress. Use kubectl to interact with clusters. Implement rolling updates and rollbacks. Understand pod lifecycle and troubleshooting.
2. **Configure networking and storage**: Set up service discovery, implement ingress controllers for external access, configure network policies for security. Provision persistent storage with PVs, PVCs, and StorageClasses.
3. **Implement auto-scaling and self-healing**: Configure HPA for automatic scaling based on metrics. Set up readiness and liveness probes for self-healing. Implement resource requests and limits for efficient scheduling.
4. **Manage secrets and configurations**: Externalize configuration using ConfigMaps and Secrets. Integrate external secret management systems. Implement proper secret rotation and access controls.
5. **Troubleshoot cluster issues**: Use kubectl debugging commands (logs, describe, exec, port-forward). Understand common failure scenarios. Monitor cluster health using metrics and logs. Debug networking issues across services.

## Projects

### Beginner Level

#### Project 1: Deploy Multi-Tier Application
**Objective**: Deploy a complete application stack on Kubernetes
- Frontend deployment with multiple replicas
- Backend API deployment
- Database StatefulSet
- ConfigMaps for configuration
- Services for inter-pod communication
- Ingress for external access

**Learning Outcomes**: Core resources, deployments, services, basic networking

#### Project 2: Kubernetes Cluster Setup (The Hard Way)
**Objective**: Set up Kubernetes cluster from scratch
- Provision VMs (using Vagrant or cloud)
- Install container runtime
- Bootstrap control plane
- Configure networking (Calico/Flannel)
- Join worker nodes
- Deploy applications

**Learning Outcomes**: Cluster architecture, components, networking fundamentals

#### Project 3: Application Health and Monitoring
**Objective**: Implement health checks and monitoring
- Liveness and readiness probes
- Resource requests and limits
- Metrics server installation
- Basic monitoring with kubectl top
- Dashboard for visualization

**Learning Outcomes**: Health checks, resource management, monitoring basics

### Intermediate Level

#### Project 4: CI/CD Pipeline on Kubernetes
**Objective**: Build a complete CI/CD pipeline
- GitOps with ArgoCD or Flux
- Automated deployments on git push
- Blue-green deployments
- Canary deployments
- Automated rollbacks
- Pipeline notifications

**Learning Outcomes**: GitOps, deployment strategies, automation

#### Project 5: Multi-Cluster Management Platform
**Objective**: Manage multiple Kubernetes clusters
- Cluster Federation
- Multi-cluster service discovery
- Cross-cluster load balancing
- Centralized monitoring and logging
- Policy enforcement across clusters

**Learning Outcomes**: Multi-cluster architectures, federation, management

#### Project 6: Kubernetes Operator Development
**Objective**: Build a custom Kubernetes operator
- Define Custom Resource Definitions (CRDs)
- Implement controller logic
- Handle resource reconciliation
- Automated backups
- Status reporting
- Integration with external systems

**Learning Outcomes**: Operators, CRDs, controller pattern, Go programming

### Advanced Level

#### Project 7: Service Mesh Implementation
**Objective**: Deploy and configure a service mesh
- Istio or Linkerd installation
- Traffic management (routing, splitting, mirroring)
- Security (mTLS, authorization policies)
- Observability (distributed tracing, metrics)
- Fault injection and chaos engineering
- Multi-cluster service mesh

**Learning Outcomes**: Service mesh, microservices patterns, observability

#### Project 8: Kubernetes Platform as a Service (PaaS)
**Objective**: Build an internal PaaS on Kubernetes
- Self-service application deployment
- Multi-tenancy with namespaces
- Resource quotas and limits
- RBAC for different user roles
- Automated DNS and certificates
- Cost tracking and chargeback
- Developer portal

**Learning Outcomes**: Platform engineering, multi-tenancy, self-service

#### Project 9: Cloud-Native Disaster Recovery System
**Objective**: Implement comprehensive DR for Kubernetes
- Automated cluster backups (Velero)
- Application-level backups
- Cross-region replication
- Automated disaster recovery drills
- RTO/RPO monitoring
- Backup validation and testing
- Documentation automation

**Learning Outcomes**: Disaster recovery, backup strategies, business continuity

## Resources

### Documentation
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes the Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [Kubernetes Patterns](https://www.redhat.com/en/resources/kubernetes-patterns-ebook)

### Books
- "Kubernetes in Action" by Marko Lukša
- "Kubernetes Up & Running" by Kelsey Hightower
- "Production Kubernetes" by Josh Rosso, Rich Lander, Alex Brand, John Harris

### Tools
- kubectl
- Helm
- kustomize
- k9s
- Lens
- ArgoCD
- Istio

### Certifications
- Certified Kubernetes Administrator (CKA)
- Certified Kubernetes Application Developer (CKAD)
- Certified Kubernetes Security Specialist (CKS)

### Practice
- [KillerCoda Kubernetes](https://killercoda.com/kubernetes)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)
- [Kubernetes Learning Environment](https://kubernetes.io/docs/tutorials/kubernetes-basics/)
