# Cloud Platforms Cheatsheet

## Quick Reference Guide for Platform Engineers

### AWS CLI Essentials

```bash
# Configuration
aws configure                              # Interactive configuration
aws configure --profile prod               # Configure named profile
aws configure list                         # Show current configuration
aws configure get region                   # Get specific setting

# Using Profiles
export AWS_PROFILE=prod                    # Set default profile
aws s3 ls --profile dev                    # Use specific profile

# EC2 (Elastic Compute Cloud)
aws ec2 describe-instances                 # List all instances
aws ec2 describe-instances --filters "Name=tag:Name,Values=web-server"
aws ec2 run-instances --image-id ami-xxx --instance-type t3.micro --key-name mykey
aws ec2 start-instances --instance-ids i-1234567890abcdef0
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
aws ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-hvm-*"
aws ec2 create-security-group --group-name my-sg --description "My SG"
aws ec2 authorize-security-group-ingress --group-id sg-xxx --protocol tcp --port 22 --cidr 0.0.0.0/0

# S3 (Simple Storage Service)
aws s3 ls                                  # List all buckets
aws s3 ls s3://bucket-name                 # List bucket contents
aws s3 cp file.txt s3://bucket-name/       # Upload file
aws s3 cp s3://bucket-name/file.txt ./     # Download file
aws s3 sync ./local s3://bucket-name/      # Sync directory
aws s3 mb s3://bucket-name                 # Create bucket
aws s3 rb s3://bucket-name --force         # Delete bucket
aws s3 presign s3://bucket-name/file.txt --expires-in 3600  # Generate presigned URL

# S3 API Operations
aws s3api list-buckets                     # List buckets with details
aws s3api get-bucket-versioning --bucket bucket-name
aws s3api put-bucket-versioning --bucket bucket-name --versioning-configuration Status=Enabled
aws s3api get-object --bucket bucket-name --key file.txt file.txt
aws s3api put-bucket-policy --bucket bucket-name --policy file://policy.json

# IAM (Identity and Access Management)
aws iam list-users                         # List IAM users
aws iam list-roles                         # List IAM roles
aws iam list-groups                        # List IAM groups
aws iam get-user                           # Get current user
aws iam create-user --user-name john       # Create user
aws iam create-access-key --user-name john # Create access key
aws iam attach-user-policy --user-name john --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
aws iam list-attached-user-policies --user-name john
aws iam get-role --role-name MyRole        # Get role details
aws iam create-role --role-name MyRole --assume-role-policy-document file://trust-policy.json

# ECS (Elastic Container Service)
aws ecs list-clusters                      # List clusters
aws ecs list-services --cluster my-cluster # List services
aws ecs list-tasks --cluster my-cluster    # List tasks
aws ecs describe-services --cluster my-cluster --services my-service
aws ecs update-service --cluster my-cluster --service my-service --desired-count 3
aws ecs describe-task-definition --task-definition my-task:1

# EKS (Elastic Kubernetes Service)
aws eks list-clusters                      # List EKS clusters
aws eks describe-cluster --name my-cluster # Cluster details
aws eks update-kubeconfig --name my-cluster --region us-east-1  # Configure kubectl
aws eks list-nodegroups --cluster-name my-cluster

# RDS (Relational Database Service)
aws rds describe-db-instances              # List RDS instances
aws rds describe-db-instances --db-instance-identifier mydb
aws rds create-db-snapshot --db-instance-identifier mydb --db-snapshot-identifier mydb-snapshot
aws rds restore-db-instance-from-db-snapshot --db-instance-identifier mydb-new --db-snapshot-identifier mydb-snapshot
aws rds modify-db-instance --db-instance-identifier mydb --backup-retention-period 7 --apply-immediately

# Lambda
aws lambda list-functions                  # List Lambda functions
aws lambda invoke --function-name my-function output.txt  # Invoke function
aws lambda update-function-code --function-name my-function --zip-file fileb://function.zip
aws lambda get-function --function-name my-function
aws lambda create-function --function-name my-function --runtime python3.9 --role arn:aws:iam::xxx:role/lambda-role --handler index.handler --zip-file fileb://function.zip

# CloudFormation
aws cloudformation list-stacks             # List stacks
aws cloudformation describe-stacks --stack-name my-stack
aws cloudformation create-stack --stack-name my-stack --template-body file://template.yaml
aws cloudformation update-stack --stack-name my-stack --template-body file://template.yaml
aws cloudformation delete-stack --stack-name my-stack
aws cloudformation validate-template --template-body file://template.yaml

# CloudWatch Logs
aws logs describe-log-groups               # List log groups
aws logs tail /aws/lambda/my-function --follow  # Tail logs
aws logs filter-log-events --log-group-name /aws/lambda/my-function --filter-pattern "ERROR"
aws logs create-log-group --log-group-name my-log-group
aws logs put-retention-policy --log-group-name my-log-group --retention-in-days 7

# CloudWatch Metrics
aws cloudwatch list-metrics                # List metrics
aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUUtilization --dimensions Name=InstanceId,Value=i-xxx --start-time 2024-01-01T00:00:00Z --end-time 2024-01-02T00:00:00Z --period 3600 --statistics Average

# VPC (Virtual Private Cloud)
aws ec2 describe-vpcs                      # List VPCs
aws ec2 describe-subnets                   # List subnets
aws ec2 describe-route-tables              # List route tables
aws ec2 describe-internet-gateways         # List internet gateways
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.1.0/24

# Route53 (DNS)
aws route53 list-hosted-zones              # List hosted zones
aws route53 list-resource-record-sets --hosted-zone-id Z123456
aws route53 change-resource-record-sets --hosted-zone-id Z123456 --change-batch file://changes.json

# Cost Explorer
aws ce get-cost-and-usage --time-period Start=2024-01-01,End=2024-01-31 --granularity MONTHLY --metrics BlendedCost
aws ce get-cost-forecast --time-period Start=2024-02-01,End=2024-02-28 --metric BLENDED_COST --granularity MONTHLY

# Systems Manager (SSM)
aws ssm get-parameter --name /my/parameter # Get parameter
aws ssm get-parameters-by-path --path /my/app --recursive
aws ssm put-parameter --name /my/parameter --value "secret" --type SecureString
aws ssm start-session --target i-xxx       # Connect to instance

# Secrets Manager
aws secretsmanager list-secrets            # List secrets
aws secretsmanager get-secret-value --secret-id my-secret
aws secretsmanager create-secret --name my-secret --secret-string '{"key":"value"}'
aws secretsmanager rotate-secret --secret-id my-secret
```

### Azure CLI Essentials

```bash
# Login and Account Management
az login                                   # Interactive login
az login --service-principal -u <app-id> -p <password> --tenant <tenant-id>
az account list                            # List subscriptions
az account set --subscription "My Subscription"
az account show                            # Show active subscription

# Resource Groups
az group list                              # List resource groups
az group create --name myResourceGroup --location eastus
az group delete --name myResourceGroup --yes --no-wait
az group show --name myResourceGroup

# Virtual Machines
az vm list                                 # List VMs
az vm list --output table                  # Table format
az vm create --resource-group myRG --name myVM --image UbuntuLTS --admin-username azureuser --generate-ssh-keys
az vm start --resource-group myRG --name myVM
az vm stop --resource-group myRG --name myVM
az vm deallocate --resource-group myRG --name myVM
az vm delete --resource-group myRG --name myVM
az vm show --resource-group myRG --name myVM
az vm list-ip-addresses --resource-group myRG --name myVM

# Storage Accounts
az storage account list                    # List storage accounts
az storage account create --name mystorageaccount --resource-group myRG --location eastus --sku Standard_LRS
az storage account keys list --resource-group myRG --account-name mystorageaccount
az storage blob upload --account-name mystorageaccount --container-name mycontainer --name myblob --file ./file.txt
az storage blob download --account-name mystorageaccount --container-name mycontainer --name myblob --file ./downloaded.txt
az storage container list --account-name mystorageaccount

# Azure Kubernetes Service (AKS)
az aks list                                # List AKS clusters
az aks create --resource-group myRG --name myAKSCluster --node-count 3 --enable-addons monitoring --generate-ssh-keys
az aks get-credentials --resource-group myRG --name myAKSCluster  # Configure kubectl
az aks scale --resource-group myRG --name myAKSCluster --node-count 5
az aks upgrade --resource-group myRG --name myAKSCluster --kubernetes-version 1.25.0
az aks delete --resource-group myRG --name myAKSCluster

# Container Registry (ACR)
az acr list                                # List container registries
az acr create --resource-group myRG --name myRegistry --sku Basic
az acr login --name myRegistry             # Login to registry
az acr repository list --name myRegistry   # List repositories
az acr repository show-tags --name myRegistry --repository myapp

# App Service
az webapp list                             # List web apps
az webapp create --resource-group myRG --plan myAppServicePlan --name myWebApp --runtime "NODE|16-lts"
az webapp start --resource-group myRG --name myWebApp
az webapp stop --resource-group myRG --name myWebApp
az webapp restart --resource-group myRG --name myWebApp
az webapp deployment source config --resource-group myRG --name myWebApp --repo-url https://github.com/user/repo --branch main

# Azure SQL Database
az sql server list                         # List SQL servers
az sql server create --resource-group myRG --name myserver --admin-user myadmin --admin-password MyP@ssw0rd
az sql db list --resource-group myRG --server myserver
az sql db create --resource-group myRG --server myserver --name mydb --service-objective S0

# Azure Functions
az functionapp list                        # List function apps
az functionapp create --resource-group myRG --consumption-plan-location eastus --runtime node --name myfunctionapp --storage-account mystorageaccount
az functionapp deployment source config --resource-group myRG --name myfunctionapp --repo-url https://github.com/user/repo --branch main

# Networking
az network vnet list                       # List virtual networks
az network vnet create --resource-group myRG --name myVNet --address-prefix 10.0.0.0/16
az network vnet subnet create --resource-group myRG --vnet-name myVNet --name mySubnet --address-prefix 10.0.1.0/24
az network nsg list                        # List network security groups
az network nsg create --resource-group myRG --name myNSG
az network nsg rule create --resource-group myRG --nsg-name myNSG --name AllowSSH --priority 1000 --source-address-prefixes '*' --destination-port-ranges 22

# Monitor
az monitor metrics list --resource /subscriptions/xxx/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM --metric "Percentage CPU"
az monitor log-analytics workspace list    # List Log Analytics workspaces
az monitor activity-log list               # List activity logs

# Key Vault
az keyvault list                           # List key vaults
az keyvault create --name myKeyVault --resource-group myRG --location eastus
az keyvault secret set --vault-name myKeyVault --name mySecret --value "secretValue"
az keyvault secret show --vault-name myKeyVault --name mySecret
az keyvault secret list --vault-name myKeyVault

# Active Directory
az ad user list                            # List users
az ad group list                           # List groups
az ad sp list                              # List service principals
az ad sp create-for-rbac --name myServicePrincipal --role Contributor

# Role-Based Access Control (RBAC)
az role assignment list                    # List role assignments
az role assignment create --assignee <user-or-sp> --role Contributor --resource-group myRG
az role definition list                    # List role definitions
```

### Google Cloud Platform (gcloud) Essentials

```bash
# Authentication and Configuration
gcloud auth login                          # Interactive login
gcloud auth application-default login      # Application default credentials
gcloud config list                         # Show configuration
gcloud config set project my-project       # Set active project
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a
gcloud config configurations list          # List configurations
gcloud config configurations create prod   # Create configuration
gcloud config configurations activate prod # Activate configuration

# Projects
gcloud projects list                       # List projects
gcloud projects create my-project --name="My Project"
gcloud projects describe my-project
gcloud projects delete my-project

# Compute Engine
gcloud compute instances list              # List instances
gcloud compute instances create my-instance --machine-type=e2-medium --image-family=ubuntu-2004-lts --image-project=ubuntu-os-cloud
gcloud compute instances start my-instance
gcloud compute instances stop my-instance
gcloud compute instances delete my-instance
gcloud compute instances describe my-instance
gcloud compute ssh my-instance             # SSH to instance
gcloud compute scp local-file my-instance:~/remote-file  # Copy file

# Compute Engine - Disk Management
gcloud compute disks list                  # List disks
gcloud compute disks create my-disk --size=100GB
gcloud compute disks snapshot my-disk --snapshot-names=my-snapshot

# Compute Engine - Firewall
gcloud compute firewall-rules list         # List firewall rules
gcloud compute firewall-rules create allow-http --allow tcp:80 --source-ranges 0.0.0.0/0
gcloud compute firewall-rules delete allow-http

# Google Kubernetes Engine (GKE)
gcloud container clusters list             # List GKE clusters
gcloud container clusters create my-cluster --num-nodes=3 --machine-type=e2-medium
gcloud container clusters get-credentials my-cluster  # Configure kubectl
gcloud container clusters resize my-cluster --num-nodes=5
gcloud container clusters upgrade my-cluster --master --cluster-version=1.25
gcloud container clusters delete my-cluster

# Cloud Storage
gsutil ls                                  # List buckets
gsutil ls gs://bucket-name                 # List bucket contents
gsutil mb gs://bucket-name                 # Create bucket
gsutil cp file.txt gs://bucket-name/       # Upload file
gsutil cp gs://bucket-name/file.txt ./     # Download file
gsutil rsync -r ./local gs://bucket-name/  # Sync directory
gsutil rm -r gs://bucket-name              # Delete bucket
gsutil iam get gs://bucket-name            # Get bucket IAM policy
gsutil iam set policy.json gs://bucket-name # Set bucket IAM policy

# Cloud SQL
gcloud sql instances list                  # List Cloud SQL instances
gcloud sql instances create my-instance --database-version=POSTGRES_13 --tier=db-f1-micro --region=us-central1
gcloud sql databases list --instance=my-instance
gcloud sql databases create mydb --instance=my-instance
gcloud sql connect my-instance --user=postgres  # Connect to instance

# Cloud Functions
gcloud functions list                      # List functions
gcloud functions deploy my-function --runtime nodejs16 --trigger-http --allow-unauthenticated --entry-point=handler --source=.
gcloud functions call my-function          # Invoke function
gcloud functions logs read my-function     # Read logs
gcloud functions delete my-function

# Cloud Run
gcloud run services list                   # List Cloud Run services
gcloud run deploy my-service --image gcr.io/my-project/my-image --platform managed --region us-central1 --allow-unauthenticated
gcloud run services describe my-service --region us-central1
gcloud run services delete my-service --region us-central1

# Container Registry (GCR)
gcloud container images list               # List images
gcloud container images list-tags gcr.io/my-project/my-image
gcloud container images delete gcr.io/my-project/my-image:tag

# Artifact Registry
gcloud artifacts repositories list         # List repositories
gcloud artifacts repositories create my-repo --repository-format=docker --location=us-central1
gcloud artifacts docker images list us-central1-docker.pkg.dev/my-project/my-repo

# IAM
gcloud iam service-accounts list           # List service accounts
gcloud iam service-accounts create my-sa --display-name="My Service Account"
gcloud iam service-accounts keys create key.json --iam-account=my-sa@my-project.iam.gserviceaccount.com
gcloud projects add-iam-policy-binding my-project --member=serviceAccount:my-sa@my-project.iam.gserviceaccount.com --role=roles/editor

# Logging
gcloud logging read "resource.type=gce_instance" --limit 10
gcloud logging read "severity>=ERROR" --limit 50 --format json

# Monitoring
gcloud monitoring dashboards list          # List dashboards
gcloud monitoring policies list            # List alerting policies

# DNS (Cloud DNS)
gcloud dns managed-zones list              # List managed zones
gcloud dns managed-zones create my-zone --dns-name=example.com. --description="My Zone"
gcloud dns record-sets list --zone=my-zone

# VPC Networking
gcloud compute networks list               # List VPC networks
gcloud compute networks create my-network --subnet-mode=custom
gcloud compute networks subnets create my-subnet --network=my-network --range=10.0.1.0/24 --region=us-central1

# Load Balancing
gcloud compute forwarding-rules list       # List forwarding rules
gcloud compute backend-services list       # List backend services
gcloud compute url-maps list               # List URL maps

# Secrets Manager
gcloud secrets list                        # List secrets
gcloud secrets create my-secret --replication-policy="automatic"
gcloud secrets versions add my-secret --data-file="/path/to/file.txt"
gcloud secrets versions access latest --secret="my-secret"

# Billing
gcloud billing accounts list               # List billing accounts
gcloud billing projects link my-project --billing-account=ACCOUNT_ID
```

### Common Cloud Services Comparison

```bash
# Compute Services
# AWS EC2 <-> Azure VM <-> GCP Compute Engine
aws ec2 describe-instances
az vm list
gcloud compute instances list

# Container Orchestration
# AWS EKS <-> Azure AKS <-> GCP GKE
aws eks list-clusters
az aks list
gcloud container clusters list

# Object Storage
# AWS S3 <-> Azure Blob Storage <-> GCP Cloud Storage
aws s3 ls
az storage container list --account-name myaccount
gsutil ls

# Serverless Functions
# AWS Lambda <-> Azure Functions <-> GCP Cloud Functions
aws lambda list-functions
az functionapp list
gcloud functions list

# Managed Kubernetes
# AWS EKS <-> Azure AKS <-> GCP GKE
eksctl create cluster
az aks create
gcloud container clusters create

# Container Registry
# AWS ECR <-> Azure ACR <-> GCP GCR/Artifact Registry
aws ecr describe-repositories
az acr list
gcloud container images list

# DNS
# AWS Route53 <-> Azure DNS <-> GCP Cloud DNS
aws route53 list-hosted-zones
az network dns zone list
gcloud dns managed-zones list

# Secrets Management
# AWS Secrets Manager <-> Azure Key Vault <-> GCP Secret Manager
aws secretsmanager list-secrets
az keyvault secret list --vault-name myvault
gcloud secrets list
```

### Multi-Cloud Tools

```bash
# Terraform (Infrastructure as Code)
terraform init
terraform plan
terraform apply
terraform destroy

# Pulumi (Infrastructure as Code)
pulumi up
pulumi destroy
pulumi stack output

# kubectl (Kubernetes)
kubectl get nodes
kubectl get pods --all-namespaces
kubectl apply -f deployment.yaml

# Docker
docker build -t myimage:tag .
docker push registry.io/myimage:tag
docker run -d -p 80:80 myimage:tag

# Helm (Kubernetes Package Manager)
helm install myapp ./chart
helm upgrade myapp ./chart
helm list
helm uninstall myapp
```

### Cost Optimization Tips

```bash
# AWS Cost Optimization
# 1. Use Reserved Instances for predictable workloads
aws ec2 describe-reserved-instances

# 2. Stop/terminate unused resources
aws ec2 describe-instances --filters "Name=instance-state-name,Values=stopped"
aws rds describe-db-instances --query 'DBInstances[?DBInstanceStatus==`stopped`]'

# 3. Use S3 Lifecycle Policies
aws s3api put-bucket-lifecycle-configuration --bucket my-bucket --lifecycle-configuration file://lifecycle.json

# 4. Enable S3 Intelligent-Tiering
aws s3api put-bucket-intelligent-tiering-configuration --bucket my-bucket --id rule1 --intelligent-tiering-configuration file://config.json

# 5. Delete unattached EBS volumes
aws ec2 describe-volumes --filters "Name=status,Values=available"

# 6. Delete old snapshots
aws ec2 describe-snapshots --owner-ids self --query 'Snapshots[?StartTime<=`2023-01-01`]'

# 7. Use Spot Instances for non-critical workloads
aws ec2 request-spot-instances --spot-price "0.05" --instance-count 1 --type "one-time" --launch-specification file://spec.json

# 8. Right-size instances using Cost Explorer
aws ce get-rightsizing-recommendation

# Azure Cost Optimization
# 1. Deallocate VMs when not in use
az vm deallocate --resource-group myRG --name myVM

# 2. Use Azure Advisor for recommendations
az advisor recommendation list

# 3. Use Azure Cost Management
az costmanagement query

# 4. Delete unused resources
az resource list --query "[?tags.environment=='dev']"

# 5. Use Azure Reserved VM Instances
az reservations reservation list

# GCP Cost Optimization
# 1. Use Committed Use Discounts
gcloud compute commitments list

# 2. Stop instances when not in use
gcloud compute instances stop my-instance

# 3. Use Preemptible VMs
gcloud compute instances create my-instance --preemptible

# 4. Delete unused disks
gcloud compute disks list --filter="users:*" --format="table(name,sizeGb,zone,users)"

# 5. Use Cloud Storage lifecycle management
gsutil lifecycle set lifecycle.json gs://bucket-name

# 6. Enable sustained use discounts (automatic)

# 7. Use labels for cost tracking
gcloud compute instances add-labels my-instance --labels=env=dev,team=platform
```

### Cloud Cost Monitoring

```bash
# AWS Cost Monitoring
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-01-31 \
  --granularity MONTHLY \
  --metrics BlendedCost \
  --group-by Type=TAG,Key=Environment

# Azure Cost Monitoring
az consumption usage list \
  --start-date 2024-01-01 \
  --end-date 2024-01-31

# GCP Cost Monitoring
gcloud billing accounts describe ACCOUNT_ID
gcloud alpha billing budgets list --billing-account=ACCOUNT_ID

# Set Budget Alerts
# AWS
aws budgets create-budget --account-id ACCOUNT_ID --budget file://budget.json

# Azure
az consumption budget create \
  --budget-name my-budget \
  --amount 1000 \
  --time-grain Monthly

# GCP
gcloud alpha billing budgets create \
  --billing-account=ACCOUNT_ID \
  --display-name="Monthly Budget" \
  --budget-amount=1000USD
```

### Cloud Security Best Practices

```bash
# Enable MFA
# AWS
aws iam enable-mfa-device --user-name john --serial-number arn:aws:iam::xxx:mfa/john --authentication-code1 123456 --authentication-code2 789012

# Azure
az ad user update --id user@example.com --force-change-password-next-login true

# GCP
# (MFA is configured in Cloud Console)

# Enable CloudTrail/Activity Logs
# AWS
aws cloudtrail create-trail --name my-trail --s3-bucket-name my-bucket
aws cloudtrail start-logging --name my-trail

# Azure
az monitor activity-log alert create --name my-alert --resource-group myRG

# GCP
gcloud logging sinks create my-sink storage.googleapis.com/my-bucket

# Encrypt at Rest
# AWS EBS
aws ec2 create-volume --size 100 --encrypted --availability-zone us-east-1a

# Azure Disk
az disk create --resource-group myRG --name myDisk --size-gb 100 --encryption-type EncryptionAtRestWithPlatformKey

# GCP Disk
gcloud compute disks create my-disk --size 100GB --type pd-standard --kms-key projects/my-project/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key

# Enable Security Scanning
# AWS
aws accessanalyzer create-analyzer --analyzer-name my-analyzer --type ACCOUNT

# Azure
az security assessment list

# GCP
gcloud scc findings list ORGANIZATION_ID
```

## Quick Tips

1. **Use tags/labels** - Tag all resources for cost tracking and organization
2. **Enable auto-scaling** - Automatically adjust capacity based on demand
3. **Use managed services** - Reduce operational overhead
4. **Implement least privilege** - Grant minimal necessary permissions
5. **Enable logging** - Track all API calls and access
6. **Use private networks** - Keep resources in private subnets
7. **Encrypt data** - Enable encryption at rest and in transit
8. **Regular backups** - Automate backup schedules
9. **Monitor costs** - Set up billing alerts and budgets
10. **Use infrastructure as code** - Manage resources with Terraform/CloudFormation

## Common CLI Patterns

```bash
# Filter and format output
aws ec2 describe-instances --query 'Reservations[*].Instances[*].[InstanceId,State.Name,Tags[?Key==`Name`].Value|[0]]' --output table

az vm list --query "[?powerState=='VM running'].{name:name, resourceGroup:resourceGroup}" --output table

gcloud compute instances list --format="table(name,zone,machineType,status)"

# Bulk operations
# AWS - Stop all instances with tag env=dev
aws ec2 describe-instances --filters "Name=tag:env,Values=dev" --query 'Reservations[*].Instances[*].InstanceId' --output text | xargs aws ec2 stop-instances --instance-ids

# Azure - Delete all resources in resource group
az resource list --resource-group myRG --query "[].id" --output tsv | xargs -L1 az resource delete --ids

# GCP - Delete all stopped instances
gcloud compute instances list --filter="status=TERMINATED" --format="value(name,zone)" | while read name zone; do gcloud compute instances delete $name --zone=$zone --quiet; done
```
