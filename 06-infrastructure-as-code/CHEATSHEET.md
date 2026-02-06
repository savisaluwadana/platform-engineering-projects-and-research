# Infrastructure as Code Cheatsheet

## Quick Reference Guide for Platform Engineers

### Terraform Basics

```bash
# Core Terraform Workflow
terraform init                    # Initialize working directory
terraform init -upgrade           # Upgrade provider plugins
terraform init -backend-config=   # Configure backend

terraform validate               # Validate configuration syntax
terraform fmt                    # Format code to canonical style
terraform fmt -recursive         # Format all files recursively

terraform plan                   # Preview infrastructure changes
terraform plan -out=plan.tfplan  # Save plan to file
terraform plan -var="key=value"  # Pass variable

terraform apply                  # Apply changes
terraform apply plan.tfplan      # Apply saved plan
terraform apply -auto-approve    # Skip confirmation prompt
terraform apply -target=resource # Apply specific resource

terraform destroy                # Destroy all resources
terraform destroy -auto-approve  # Destroy without confirmation
terraform destroy -target=       # Destroy specific resource

terraform show                   # Show current state
terraform show -json             # Show state in JSON format
terraform output                 # Show all outputs
terraform output variable_name   # Show specific output
```

### Terraform State Management

```bash
# State Operations
terraform state list             # List resources in state
terraform state show resource    # Show detailed resource info
terraform state pull             # Pull remote state
terraform state push             # Push state to remote

# State Manipulation (Use with caution!)
terraform state mv old new       # Rename resource
terraform state rm resource      # Remove resource from state
terraform state replace-provider # Replace provider

# Workspace Management
terraform workspace list         # List workspaces
terraform workspace new dev      # Create new workspace
terraform workspace select dev   # Switch workspace
terraform workspace delete dev   # Delete workspace

# State Locking
terraform force-unlock LOCK_ID   # Force unlock state
```

### Terraform Configuration

```hcl
# Variables
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
  
  validation {
    condition     = can(regex("^us-", var.region))
    error_message = "Region must be in US."
  }
}

variable "instance_count" {
  type    = number
  default = 2
}

variable "tags" {
  type = map(string)
  default = {
    Environment = "dev"
    Project     = "demo"
  }
}

variable "subnets" {
  type = list(string)
}

# Outputs
output "instance_id" {
  description = "EC2 instance ID"
  value       = aws_instance.web.id
  sensitive   = false
}

output "private_ips" {
  value = aws_instance.web[*].private_ip
}

# Locals
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
  name_prefix = "${var.project}-${var.environment}"
}

# Data Sources
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# Resources
resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  tags = merge(
    local.common_tags,
    {
      Name = "${local.name_prefix}-web-${count.index + 1}"
    }
  )
  
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags]
  }
}

# Provisioners
resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo ${aws_instance.web.private_ip} >> private_ips.txt"
  }
  
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
    ]
  }
}
```

### Terraform Modules

```hcl
# Module Usage
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.14.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = false
  
  tags = local.common_tags
}

# Module Outputs
output "vpc_id" {
  value = module.vpc.vpc_id
}

# Module Structure
# modules/ec2-instance/
# ├── main.tf
# ├── variables.tf
# ├── outputs.tf
# └── README.md

# modules/ec2-instance/main.tf
resource "aws_instance" "this" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = var.subnet_id
  
  tags = var.tags
}

# modules/ec2-instance/variables.tf
variable "ami_id" {
  description = "AMI ID for the instance"
  type        = string
}

# modules/ec2-instance/outputs.tf
output "instance_id" {
  value = aws_instance.this.id
}
```

### Terraform Backend Configuration

```hcl
# S3 Backend
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
    
    # Optional
    profile        = "default"
    role_arn       = "arn:aws:iam::ACCOUNT:role/TerraformRole"
  }
}

# Azure Backend
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-rg"
    storage_account_name = "tfstate"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

# Remote Backend (Terraform Cloud)
terraform {
  backend "remote" {
    organization = "my-org"
    
    workspaces {
      name = "my-workspace"
    }
  }
}
```

### Ansible Basics

```bash
# Inventory Management
ansible all --list-hosts                    # List all hosts
ansible webservers --list-hosts             # List group hosts
ansible-inventory --list                    # Show inventory in JSON
ansible-inventory --graph                   # Show inventory tree

# Ad-hoc Commands
ansible all -m ping                         # Ping all hosts
ansible all -m setup                        # Gather facts
ansible webservers -m shell -a "uptime"     # Run command
ansible all -m copy -a "src=/file dest=/tmp/" # Copy file
ansible all -m apt -a "name=nginx state=present" -b # Install package

# Playbook Execution
ansible-playbook playbook.yml               # Run playbook
ansible-playbook playbook.yml --check       # Dry run
ansible-playbook playbook.yml --diff        # Show differences
ansible-playbook playbook.yml -v            # Verbose (-vvv for more)
ansible-playbook playbook.yml --tags web    # Run specific tags
ansible-playbook playbook.yml --skip-tags db # Skip tags
ansible-playbook playbook.yml --limit web01 # Run on specific hosts
ansible-playbook playbook.yml -e "var=value" # Extra variables

# Vault (Secrets Management)
ansible-vault create secrets.yml            # Create encrypted file
ansible-vault edit secrets.yml              # Edit encrypted file
ansible-vault encrypt file.yml              # Encrypt existing file
ansible-vault decrypt file.yml              # Decrypt file
ansible-vault view secrets.yml              # View encrypted file
ansible-playbook playbook.yml --ask-vault-pass # Prompt for password
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass

# Roles
ansible-galaxy init role_name               # Create role structure
ansible-galaxy install author.role_name     # Install role
ansible-galaxy list                         # List installed roles
ansible-galaxy search nginx                 # Search for roles
```

### Ansible Inventory

```ini
# INI Format (inventory/hosts)
[webservers]
web01.example.com ansible_host=192.168.1.10
web02.example.com ansible_host=192.168.1.11

[dbservers]
db01.example.com ansible_host=192.168.1.20

[production:children]
webservers
dbservers

[production:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/prod.pem
ansible_python_interpreter=/usr/bin/python3

[all:vars]
ansible_connection=ssh
```

```yaml
# YAML Format (inventory/hosts.yml)
all:
  children:
    webservers:
      hosts:
        web01.example.com:
          ansible_host: 192.168.1.10
        web02.example.com:
          ansible_host: 192.168.1.11
      vars:
        http_port: 80
        
    dbservers:
      hosts:
        db01.example.com:
          ansible_host: 192.168.1.20
      vars:
        db_port: 5432
        
    production:
      children:
        - webservers
        - dbservers
      vars:
        ansible_user: ubuntu
        ansible_ssh_private_key_file: ~/.ssh/prod.pem
```

### Ansible Playbooks

```yaml
---
# playbook.yml - Complete Example
- name: Configure web servers
  hosts: webservers
  become: yes
  
  vars:
    http_port: 80
    app_path: /var/www/html
    
  vars_files:
    - vars/main.yml
    
  pre_tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
      changed_when: false
      
  tasks:
    - name: Install nginx
      apt:
        name: nginx
        state: present
      notify: restart nginx
      
    - name: Copy nginx config
      template:
        src: templates/nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        owner: root
        group: root
        mode: '0644'
      notify: restart nginx
      
    - name: Deploy application
      copy:
        src: "{{ item }}"
        dest: "{{ app_path }}/"
      loop:
        - index.html
        - style.css
        - app.js
      tags: deploy
      
    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
        
  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
        
  post_tasks:
    - name: Verify nginx
      uri:
        url: "http://{{ inventory_hostname }}"
        status_code: 200
```

### Ansible Roles Structure

```
roles/
└── webserver/
    ├── defaults/
    │   └── main.yml          # Default variables
    ├── files/
    │   └── index.html        # Static files
    ├── handlers/
    │   └── main.yml          # Handlers
    ├── meta/
    │   └── main.yml          # Role metadata
    ├── tasks/
    │   └── main.yml          # Main tasks
    ├── templates/
    │   └── nginx.conf.j2     # Jinja2 templates
    ├── tests/
    │   └── test.yml          # Test playbook
    └── vars/
        └── main.yml          # Role variables
```

```yaml
# roles/webserver/tasks/main.yml
---
- name: Include OS-specific variables
  include_vars: "{{ ansible_os_family }}.yml"

- name: Install web server
  package:
    name: "{{ web_server_package }}"
    state: present
    
- name: Configure web server
  template:
    src: config.j2
    dest: "{{ web_server_config_path }}"
  notify: restart web server
```

### Common IaC Patterns

```hcl
# 1. Resource Naming Convention
resource "aws_instance" "web" {
  tags = {
    Name = "${var.environment}-${var.project}-web-${count.index + 1}"
  }
}

# 2. Conditional Resources
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0
}

# 3. Dynamic Blocks
resource "aws_security_group" "web" {
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

# 4. For Expressions
locals {
  instance_ips = {
    for instance in aws_instance.web:
    instance.tags.Name => instance.private_ip
  }
}

# 5. Resource Dependencies
resource "aws_eip" "web" {
  instance   = aws_instance.web.id
  depends_on = [aws_internet_gateway.main]
}
```

### Terraform Best Practices

```bash
# 1. Use Version Constraints
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
}

# 2. Use Remote State
terraform {
  backend "s3" {
    # Configuration
  }
}

# 3. Use .gitignore
# .gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
.terraform.lock.hcl

# 4. Organize Files
# project/
# ├── main.tf          # Main resources
# ├── variables.tf     # Input variables
# ├── outputs.tf       # Outputs
# ├── providers.tf     # Provider configuration
# ├── backend.tf       # Backend configuration
# ├── versions.tf      # Version constraints
# └── terraform.tfvars # Variable values (don't commit!)

# 5. Use Modules for Reusability
module "vpc" {
  source = "./modules/vpc"
}

# 6. Use Data Sources
data "aws_ami" "latest" {
  most_recent = true
}

# 7. Implement Lifecycle Rules
lifecycle {
  create_before_destroy = true
  prevent_destroy       = true
}

# 8. Use Variables for Flexibility
variable "instance_type" {
  type    = string
  default = "t3.micro"
}
```

### Ansible Best Practices

```yaml
# 1. Use Roles for Organization
- hosts: webservers
  roles:
    - common
    - nginx
    - application

# 2. Use Variables Wisely
# Group vars: group_vars/webservers.yml
# Host vars: host_vars/web01.yml

# 3. Use Templates for Configuration
- name: Configure nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf

# 4. Use Handlers for Services
handlers:
  - name: restart nginx
    service:
      name: nginx
      state: restarted

# 5. Use Tags for Selective Execution
- name: Deploy application
  copy:
    src: app.tar.gz
    dest: /tmp/
  tags: deploy

# 6. Use Check Mode
# Run with: ansible-playbook playbook.yml --check

# 7. Use Vault for Secrets
# Create: ansible-vault create secrets.yml
# Use in playbook:
vars_files:
  - secrets.yml

# 8. Use Idempotency
- name: Ensure nginx is installed
  apt:
    name: nginx
    state: present  # Not 'latest' for idempotency

# 9. Use Blocks for Error Handling
- block:
    - name: Risky task
      command: /path/to/command
  rescue:
    - name: Handle error
      debug:
        msg: "Task failed, handling..."
  always:
    - name: Cleanup
      file:
        path: /tmp/temp
        state: absent

# 10. Use Assertions
- name: Check prerequisites
  assert:
    that:
      - ansible_os_family == "Debian"
      - ansible_python_version is version('3.6', '>=')
    fail_msg: "Prerequisites not met"
```

### Infrastructure Testing

```bash
# Terraform Testing
terraform fmt -check              # Check formatting
terraform validate                # Validate syntax
terraform plan -detailed-exitcode # Exit code 2 if changes

# TFLint - Terraform Linting
tflint --init
tflint

# Checkov - Security Scanning
checkov -d .
checkov -f main.tf

# Terraform Compliance
terraform-compliance -f compliance/ -p plan.json

# Ansible Testing
ansible-playbook playbook.yml --syntax-check  # Syntax check
ansible-lint playbook.yml                     # Linting
molecule test                                 # Full testing with Molecule

# Kitchen + InSpec (Infrastructure Testing)
kitchen test
```

### Troubleshooting

```bash
# Terraform Debugging
export TF_LOG=DEBUG                # Enable debug logging
export TF_LOG_PATH=terraform.log   # Log to file
terraform console                  # Interactive console

# Terraform Graph
terraform graph | dot -Tsvg > graph.svg

# Ansible Debugging
ansible-playbook playbook.yml -vvv          # Verbose output
ansible-playbook playbook.yml --step        # Step through tasks
ansible-playbook playbook.yml --start-at-task="task name"

# Debug Module
- name: Debug variable
  debug:
    var: my_variable
    verbosity: 2

# Register and Debug
- name: Run command
  command: whoami
  register: result

- name: Show result
  debug:
    var: result
```

## Quick Tips

1. **Always use version control** - Track all IaC code in Git
2. **Use state locking** - Prevent concurrent modifications
3. **Plan before apply** - Review changes before applying
4. **Use modules** - Promote code reuse and consistency
5. **Encrypt secrets** - Never commit plain text secrets
6. **Test in non-prod first** - Validate changes in lower environments
7. **Document your code** - Add descriptions and comments
8. **Use naming conventions** - Consistent resource naming
9. **Implement RBAC** - Control access to state and infrastructure
10. **Monitor state files** - Back up and version state files
