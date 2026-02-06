# Cloud Platforms (AWS, Azure, GCP)

## Theory

### Core Concepts

#### Cloud Computing Fundamentals
Cloud platforms provide on-demand access to computing resources over the internet:

**Service Models**:
- **IaaS (Infrastructure as a Service)**: Virtual machines, storage, networking. You manage OS and applications (EC2, Azure VMs, Compute Engine).
- **PaaS (Platform as a Service)**: Managed runtime environments. You manage applications and data (Elastic Beanstalk, App Service, App Engine).
- **SaaS (Software as a Service)**: Fully managed applications. You use the software (Gmail, Salesforce, Office 365).
- **FaaS (Function as a Service)**: Serverless compute. You write functions; platform handles execution (Lambda, Azure Functions, Cloud Functions).

**Deployment Models**:
- **Public Cloud**: Shared infrastructure, pay-as-you-go (AWS, Azure, GCP)
- **Private Cloud**: Dedicated infrastructure for single organization
- **Hybrid Cloud**: Combination of public and private clouds
- **Multi-Cloud**: Using multiple public cloud providers

**Benefits**: Elasticity, pay-per-use, global reach, managed services, rapid innovation

#### Compute Services: Running Your Workloads
Cloud providers offer various compute options for different use cases:

**Virtual Machines**:
- **AWS EC2**: Elastic Compute Cloud with hundreds of instance types optimized for compute, memory, storage, or GPU workloads
- **Azure Virtual Machines**: Windows and Linux VMs with various sizes and capabilities
- **GCP Compute Engine**: Customizable VMs with per-second billing and sustained use discounts

**Instance Types**: General purpose (balanced CPU/memory), compute-optimized (high CPU), memory-optimized (high RAM), storage-optimized (high I/O), GPU instances (ML/graphics)

**Container Services**:
- **AWS ECS/Fargate**: Container orchestration (ECS with EC2 or Fargate for serverless)
- **Azure AKS**: Managed Kubernetes service
- **GCP GKE**: Google Kubernetes Engine with autopilot mode

**Serverless Compute**:
- **AWS Lambda**: Event-driven functions, pay per execution, automatic scaling
- **Azure Functions**: Serverless compute with multiple triggers
- **Google Cloud Functions**: Lightweight, event-driven functions

**Choosing Compute**:
- VMs for full control and legacy apps
- Containers for microservices and portability
- Serverless for event-driven, variable workloads

#### Storage Services: Persisting Your Data
Cloud storage services cater to different data types and access patterns:

**Object Storage** (unstructured data, files, backups):
- **AWS S3**: Industry-leading object storage with 11 9's durability. Storage classes for different access patterns (Standard, IA, Glacier, Deep Archive)
- **Azure Blob Storage**: Scalable object storage with hot, cool, and archive tiers
- **GCP Cloud Storage**: Unified object storage with automatic storage class transitions

**Block Storage** (VM disks, databases):
- **AWS EBS**: Elastic Block Store with SSD and HDD options, snapshots for backup
- **Azure Managed Disks**: Persistent disks with various performance tiers
- **GCP Persistent Disks**: Durable block storage with automatic replication

**File Storage** (shared file systems):
- **AWS EFS**: Elastic File System, fully managed NFS
- **Azure Files**: SMB and NFS file shares in the cloud
- **GCP Filestore**: High-performance file storage for GKE and Compute Engine

**Storage Features**:
- Lifecycle policies (automatic tier transitions)
- Versioning (keep multiple versions)
- Replication (cross-region for DR)
- Encryption (at rest and in transit)

#### Networking: Connecting Cloud Resources
Cloud networking provides secure, scalable connectivity:

**Virtual Private Cloud (VPC)**:
- Isolated network environments with your own IP address ranges
- **Subnets**: Divide VPC into public (internet-facing) and private (internal) segments
- **Route Tables**: Control traffic routing between subnets and to internet
- **Internet Gateway**: Enable internet access for public subnets
- **NAT Gateway**: Allow private subnets to access internet without exposing them

**Security**:
- **Security Groups** (AWS/Azure): Stateful firewalls at instance level
- **Network ACLs** (AWS): Stateless firewalls at subnet level
- **Network Security Groups** (Azure): Both instance and subnet level
- **Firewall Rules** (GCP): VPC-level firewall configuration

**Load Balancing**:
- **Application Load Balancer**: Layer 7 (HTTP/HTTPS) with path-based routing
- **Network Load Balancer**: Layer 4 (TCP/UDP) with extreme performance
- **Classic/Gateway Load Balancer**: Legacy and advanced use cases

**Connectivity**:
- **VPN**: Encrypted connection between on-premises and cloud
- **Direct Connect/ExpressRoute/Cloud Interconnect**: Dedicated private connection
- **VPC Peering**: Connect VPCs within or across regions
- **Transit Gateway**: Hub for connecting multiple VPCs and on-premises networks

Proper network design ensures security, performance, and cost efficiency.

#### Database Services: Managed Data Stores
Cloud providers offer numerous managed database services:

**Relational Databases**:
- **AWS RDS**: Managed MySQL, PostgreSQL, MariaDB, Oracle, SQL Server. Automated backups, patching, replication
- **Azure SQL Database**: Fully managed SQL Server with serverless option
- **Google Cloud SQL**: Managed MySQL, PostgreSQL, SQL Server

**NoSQL Databases**:
- **AWS DynamoDB**: Fast, flexible NoSQL key-value and document database. Single-digit millisecond latency at any scale
- **Azure Cosmos DB**: Globally distributed, multi-model database with multiple APIs (SQL, MongoDB, Cassandra, Gremlin)
- **GCP Firestore/Datastore**: Document databases for mobile/web apps

**Data Warehouses**:
- **AWS Redshift**: Petabyte-scale data warehouse for analytics
- **Azure Synapse Analytics**: Analytics service combining data warehouse and big data
- **Google BigQuery**: Serverless, highly scalable data warehouse

**Caching**:
- **AWS ElastiCache**: Redis or Memcached for in-memory caching
- **Azure Cache for Redis**: Fully managed Redis
- **GCP Memorystore**: Redis and Memcached services

**Database Selection**: Consider data model (relational vs NoSQL), scalability needs, consistency requirements, and cost.

#### Identity and Access Management: Securing Access
IAM controls who can access what in your cloud environment:

**AWS IAM**:
- **Users**: Individual identities for people
- **Groups**: Collections of users with shared permissions
- **Roles**: Temporary credentials for services or federated users
- **Policies**: JSON documents defining permissions (Allow/Deny actions on resources)
- **MFA**: Multi-factor authentication for enhanced security

**Azure IAM**:
- **Azure Active Directory**: Identity platform integrating with Office 365
- **Service Principals**: Identities for applications and services
- **Managed Identities**: Automatic credential management for Azure services
- **RBAC**: Role-Based Access Control with built-in and custom roles

**GCP IAM**:
- **Members**: Google accounts, service accounts, groups, domains
- **Roles**: Predefined (curated by Google) or custom (user-defined)
- **Policies**: Bind roles to members on resources

**Best Practices**:
- Principle of least privilege (grant minimum required permissions)
- Use roles instead of long-term credentials
- Enable MFA for all users
- Rotate credentials regularly
- Audit access with logging and monitoring

#### Monitoring and Observability: Understanding Your Systems
Cloud monitoring provides insights into application and infrastructure health:

**AWS CloudWatch**:
- Metrics collection from AWS services and custom applications
- Logs aggregation and analysis
- Alarms for automated responses
- Dashboards for visualization
- Application Insights for distributed tracing

**Azure Monitor**:
- Unified monitoring for apps, infrastructure, and networks
- Log Analytics for querying and analysis
- Application Insights for APM
- Azure Sentinel for SIEM capabilities

**Google Cloud Monitoring (formerly Stackdriver)**:
- Metrics, logs, and traces in one platform
- Integration with open-source tools (Prometheus, Grafana)
- Error Reporting and Debugging
- Cloud Trace for distributed tracing

**Key Metrics**:
- CPU, memory, disk, network utilization
- Request rates and latencies
- Error rates and types
- Custom business metrics

Effective monitoring enables proactive issue detection and performance optimization.

#### Serverless Architecture: Focus on Code, Not Infrastructure
Serverless abstracts infrastructure management, letting you focus on business logic:

**Benefits**:
- No server management (automatic scaling, patching, HA)
- Pay per use (no idle resource costs)
- Automatic scaling (from zero to thousands of requests)
- Built-in fault tolerance

**Serverless Services**:
- **Compute**: Lambda, Azure Functions, Cloud Functions
- **Storage**: S3, Blob Storage, Cloud Storage
- **Databases**: DynamoDB, Cosmos DB, Firestore
- **API**: API Gateway, Azure API Management, Cloud Endpoints
- **Integration**: EventBridge, Event Grid, Cloud Pub/Sub

**Use Cases**:
- Event-driven processing (file uploads, database changes)
- API backends for mobile/web apps
- Scheduled tasks (cron jobs)
- Real-time stream processing
- Chatbots and Alexa skills

**Considerations**:
- Cold start latency (first invocation delay)
- Execution time limits (typically 15 minutes)
- State management (functions are stateless)
- Vendor lock-in (platform-specific APIs)

#### Cost Management: Optimizing Cloud Spend
Cloud costs can spiral without proper management:

**Cost Components**:
- Compute (per hour or second)
- Storage (per GB per month)
- Data transfer (egress charges)
- API calls/requests
- Additional services (backups, monitoring, support)

**Cost Optimization Strategies**:
- **Right-sizing**: Match instance types to workload requirements
- **Reserved Instances/Savings Plans**: Commit to usage for discounts (up to 72%)
- **Spot Instances**: Use spare capacity at steep discounts (up to 90%) for fault-tolerant workloads
- **Auto-scaling**: Scale down during low usage periods
- **Lifecycle Policies**: Move old data to cheaper storage tiers
- **Resource Tagging**: Track costs by project, team, environment
- **Budget Alerts**: Get notified before overspending
- **Resource Cleanup**: Delete unused resources (unattached volumes, old snapshots)

**FinOps**: Bring financial accountability to cloud spending through culture, tools, and processes.

### Key Skills
1. **Design cloud architecture**: Create scalable, resilient architectures using cloud-native services. Apply Well-Architected Framework principles (security, reliability, performance efficiency, cost optimization, operational excellence). Design for failure with redundancy and fault tolerance.
2. **Implement security best practices**: Configure IAM policies following least privilege. Encrypt data at rest and in transit. Implement network segmentation with VPCs and security groups. Enable logging and monitoring for security events. Use secrets management services.
3. **Optimize costs and performance**: Right-size resources based on actual usage. Use reserved instances and savings plans. Implement auto-scaling for variable workloads. Choose appropriate storage tiers. Monitor and eliminate waste. Balance cost with performance requirements.
4. **Manage multi-cloud environments**: Understand service equivalents across providers. Use cloud-agnostic tools (Terraform, Kubernetes). Implement unified monitoring and logging. Design for portability. Avoid vendor lock-in where possible.
5. **Implement disaster recovery**: Design for high availability across availability zones. Implement cross-region replication for critical data. Automate backup and restore processes. Test DR procedures regularly. Define and measure RTO/RPO objectives.

## Projects

### Beginner Level

#### Project 1: Three-Tier Web Application
**Objective**: Deploy a scalable web application on cloud
- Web tier with auto-scaling
- Application tier with load balancing
- Database tier with replication
- CDN for static assets
- SSL/TLS certificates
- Monitoring and logging

**Learning Outcomes**: Cloud architecture, high availability, security

#### Project 2: Serverless API Backend
**Objective**: Build a serverless REST API
- API Gateway
- Lambda/Cloud Functions
- DynamoDB/Firestore
- Authentication (Cognito/Auth0)
- Rate limiting
- API documentation

**Learning Outcomes**: Serverless architecture, API design, managed services

#### Project 3: Cloud Storage and CDN Solution
**Objective**: Implement a content delivery system
- Object storage (S3/Blob/GCS)
- CDN configuration
- Automated image optimization
- Lifecycle policies
- Access control
- Backup and versioning

**Learning Outcomes**: Storage services, CDN, optimization

### Intermediate Level

#### Project 4: Multi-Region High Availability System
**Objective**: Design a globally distributed application
- Multi-region deployment
- Global load balancing
- Database replication across regions
- Disaster recovery setup
- Automated failover
- Health checks and monitoring

**Learning Outcomes**: Global architecture, disaster recovery, resilience

#### Project 5: Cloud-Native Microservices Platform
**Objective**: Build a complete microservices ecosystem
- Container orchestration (EKS/AKS/GKE)
- Service mesh (Istio/Linkerd)
- API Gateway
- Service discovery
- Distributed tracing
- Centralized logging

**Learning Outcomes**: Microservices, cloud-native, observability

#### Project 6: Cloud Data Pipeline
**Objective**: Create a data processing pipeline
- Data ingestion (Kinesis/Event Hubs/Pub/Sub)
- Stream processing
- Batch processing
- Data warehouse (Redshift/Synapse/BigQuery)
- BI dashboards
- Data governance

**Learning Outcomes**: Data engineering, ETL, analytics

### Advanced Level

#### Project 7: Multi-Cloud Abstraction Layer
**Objective**: Build a cloud-agnostic platform
- Unified API for AWS/Azure/GCP
- Automated cloud selection based on cost/performance
- Cross-cloud networking
- Unified monitoring and logging
- Cost optimization across clouds
- Disaster recovery across providers

**Learning Outcomes**: Multi-cloud strategy, abstraction, vendor independence

#### Project 8: Cloud Cost Optimization Platform
**Objective**: Create an intelligent cost management system
- Resource utilization analysis
- Right-sizing recommendations
- Reserved instance/savings plan optimization
- Spot instance management
- Automated resource cleanup
- Cost anomaly detection
- Showback/chargeback system

**Learning Outcomes**: Cost optimization, FinOps, resource management

#### Project 9: Cloud Security and Compliance Platform
**Objective**: Implement comprehensive cloud security
- Security posture management (CSPM)
- Compliance monitoring (SOC2, HIPAA, PCI-DSS)
- Automated remediation
- Threat detection and response
- Secrets management
- Data encryption and key management
- Security audit automation

**Learning Outcomes**: Cloud security, compliance, threat detection

## Resources

### Documentation
- [AWS Documentation](https://docs.aws.amazon.com/)
- [Azure Documentation](https://docs.microsoft.com/en-us/azure/)
- [GCP Documentation](https://cloud.google.com/docs)

### Books
- "AWS Certified Solutions Architect Study Guide"
- "Microsoft Azure Architect Technologies and Design" 
- "Google Cloud Platform in Action"

### Tools
- AWS CLI/SDK
- Azure CLI/SDK
- gcloud CLI/SDK
- Terraform
- Cloud formation
- Cost management tools

### Training Platforms
- AWS Skill Builder
- Microsoft Learn
- Google Cloud Skills Boost
- A Cloud Guru
- Linux Academy

### Certifications

**AWS**
- Solutions Architect Associate/Professional
- Developer Associate
- SysOps Administrator Associate
- DevOps Engineer Professional

**Azure**
- Azure Administrator Associate
- Azure Solutions Architect Expert
- Azure DevOps Engineer Expert

**GCP**
- Associate Cloud Engineer
- Professional Cloud Architect
- Professional Cloud DevOps Engineer

### Best Practices
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Azure Well-Architected Framework](https://docs.microsoft.com/en-us/azure/architecture/framework/)
- [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)
