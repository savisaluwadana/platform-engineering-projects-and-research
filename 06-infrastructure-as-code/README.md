# Infrastructure as Code (IaC)

## Theory

### Core Concepts

#### Infrastructure as Code Principles: Modern Infrastructure Management
Infrastructure as Code (IaC) treats infrastructure configuration as software, enabling version control, testing, and automation:

**Declarative vs Imperative**:
- **Declarative**: Describe the desired end state; the tool determines how to achieve it (Terraform, CloudFormation, Kubernetes manifests). Focus on "what" you want.
- **Imperative**: Specify exact steps to achieve the desired state (Ansible, scripts, Chef recipes). Focus on "how" to get there.
- **Hybrid**: Some tools combine both approaches (Ansible can be declarative with modules that ensure state).

**Idempotency**: Running the same IaC code multiple times produces the same result without unintended side effects. Essential for reliability and safe re-execution.

**Version Control**: Store IaC in Git to:
- Track changes over time
- Enable code review for infrastructure changes
- Support rollback to previous configurations
- Audit who changed what and when
- Enable collaboration across teams

**Benefits**:
- Reproducibility: Create identical environments consistently
- Speed: Provision infrastructure in minutes, not days
- Documentation: Code serves as living documentation
- Cost Reduction: Automate repetitive tasks
- Risk Reduction: Test changes before production deployment

IaC transforms infrastructure management from manual, error-prone processes to automated, tested, and reliable workflows.

#### Terraform: Cloud-Agnostic Infrastructure Provisioning
Terraform is the leading IaC tool using HashiCorp Configuration Language (HCL):

**Core Components**:
- **HCL Syntax**: Declarative language with blocks, arguments, and expressions. Readable and writable by humans while being machine-parsable.
- **Providers**: Plugins enabling Terraform to interact with APIs (AWS, Azure, GCP, Kubernetes, GitHub, etc.). Each provider has resources and data sources.
- **Resources**: Infrastructure components to create/manage (VMs, networks, databases, DNS records). Defined with `resource` blocks.
- **Data Sources**: Query existing infrastructure or external data without managing it. Useful for referencing existing resources.
- **Variables**: Parameterize configurations for reusability. Support types (string, number, bool, list, map, object), defaults, and validation.
- **Outputs**: Export values from modules or configurations. Used for displaying information or passing data between modules.
- **Modules**: Reusable Terraform configurations. Promote DRY principle and enable sharing of common patterns.
- **Functions**: Built-in functions for string manipulation, collection operations, encoding, filesystem operations, and more.

**Terraform Workflow**:
1. **Write**: Define resources in `.tf` files
2. **Init**: Download provider plugins with `terraform init`
3. **Plan**: Preview changes with `terraform plan`
4. **Apply**: Create/modify infrastructure with `terraform apply`
5. **Destroy**: Remove infrastructure with `terraform destroy` (when needed)

Terraform's cloud-agnostic approach enables multi-cloud strategies and prevents vendor lock-in.

#### State Management: Tracking Infrastructure Reality
Terraform state is the source of truth for managed infrastructure:

- **State File**: JSON file mapping Terraform configuration to real-world resources. Contains resource IDs, attributes, and metadata.
- **Remote State**: Store state in remote backends (S3, Azure Blob, Terraform Cloud, Consul) for:
  - Team collaboration (shared access)
  - Enhanced security (encryption at rest)
  - State locking (prevent concurrent modifications)
  - State versioning (rollback capability)
- **State Locking**: Prevents concurrent operations that could corrupt state. Supported by backends like S3 with DynamoDB, Azure Blob, GCS.
- **Workspaces**: Manage multiple states within a single configuration (for environments like dev, staging, prod). Alternative to separate directories.
- **State Commands**: `terraform state list`, `show`, `mv`, `rm` for state manipulation.
- **Sensitive Data**: State files contain sensitive data (passwords, keys). Encrypt at rest and restrict access.

Proper state management is critical for Terraform's reliability and team collaboration.

#### Ansible: Configuration Management and Automation
Ansible is an agentless automation tool using SSH for configuration management:

**Architecture**:
- **Agentless**: No software to install on managed nodes, just SSH access
- **Push-based**: Control node pushes configurations to targets (vs pull-based like Puppet/Chef)
- **YAML Syntax**: Human-readable playbooks defining tasks and configurations

**Key Concepts**:
- **Playbooks**: YAML files defining automation tasks. Describe policies and procedures in code.
- **Roles**: Organize playbooks into reusable components with defined structure (tasks, handlers, files, templates, variables).
- **Inventory**: Lists of hosts organized into groups. Can be static (INI/YAML files) or dynamic (scripts querying cloud APIs).
- **Modules**: Reusable units of automation (file, copy, yum, service, etc.). Idempotent by design.
- **Tasks**: Individual units of work using modules. Combined into playbooks.
- **Handlers**: Special tasks triggered by notifications, typically for service restarts.
- **Templates**: Jinja2 templates for generating configuration files dynamically.
- **Variables**: Parameterize playbooks for flexibility. Support precedence hierarchy.
- **Ansible Galaxy**: Repository of community roles and collections.
- **Ansible Vault**: Encrypt sensitive data within playbooks.

**Use Cases**:
- Server provisioning and configuration
- Application deployment
- Security hardening
- Orchestration of complex workflows

Ansible's simplicity and agentless nature make it ideal for configuration management and orchestration.

#### Alternative Configuration Management Tools
Understanding the ecosystem helps choose the right tool:

- **Puppet**: Declarative, pull-based configuration management with custom DSL. Agent-based with master-agent architecture. Strong for large-scale deployments with complex dependencies.
- **Chef**: Imperative configuration management using Ruby DSL. Agent-based (Chef Client) communicating with Chef Server. Excellent for infrastructure as code with test-driven development.
- **SaltStack**: Event-driven automation using Python. Fast execution with ZeroMQ messaging. Supports both agent-based (minions) and agentless (SSH) modes.
- **Comparison**: Ansible excels in simplicity and agentless approach; Puppet/Chef offer more powerful abstractions for large enterprises; SaltStack provides high performance for event-driven automation.

#### CloudFormation: AWS-Native Infrastructure as Code
AWS CloudFormation is Amazon's IaC service:

- **Templates**: JSON or YAML files defining AWS resources and configurations
- **Stacks**: Collection of AWS resources managed as a single unit. Create, update, or delete entire stacks.
- **Change Sets**: Preview changes before applying to production stacks
- **Drift Detection**: Identify manual changes made outside CloudFormation
- **StackSets**: Deploy stacks across multiple accounts and regions
- **Advantages**: Deep AWS integration, no cost for CloudFormation itself, AWS-native features
- **Limitations**: AWS-only, less flexible than Terraform, verbose syntax

CloudFormation is ideal for AWS-exclusive environments requiring deep service integration.

#### Pulumi: Programming Language-Based IaC
Pulumi enables IaC using general-purpose programming languages:

- **Supported Languages**: TypeScript, Python, Go, C#, Java - use familiar languages instead of custom DSLs
- **Real Programming**: Use loops, conditionals, functions, classes, and libraries
- **Type Safety**: Catch errors at compile time with IDE support
- **State Management**: Similar to Terraform with backend options
- **Pulumi Service**: SaaS for state management, secrets, and CI/CD integration
- **Advantages**: Leverage existing language skills, powerful abstractions, rich ecosystem
- **Considerations**: Steeper learning curve than declarative tools, requires programming knowledge

Pulumi bridges software development and infrastructure management.

#### Testing Infrastructure Code: Quality Assurance
Testing IaC ensures reliability and catches issues before production:

**Testing Levels**:
- **Static Analysis**: Scan code without execution
  - **tfsec**: Security scanning for Terraform
  - **Checkov**: Policy-as-code for Terraform, CloudFormation, Kubernetes
  - **TFLint**: Linting for Terraform to catch errors and enforce best practices
- **Unit Tests**: Test individual modules in isolation
  - **Terratest**: Go-based testing framework for Terraform
  - **kitchen-terraform**: Integration with Test Kitchen
- **Integration Tests**: Test infrastructure after provisioning
  - Verify resources created correctly
  - Test connectivity and functionality
  - Validate security configurations
- **Compliance Tests**: Ensure adherence to policies
  - **InSpec**: Compliance testing framework
  - **Open Policy Agent (OPA)**: Policy engine for cloud-native environments
- **Contract Tests**: Verify module interfaces and outputs

**Testing Best Practices**:
- Test modules in isolation before integration
- Use temporary environments for testing
- Automate testing in CI/CD pipelines
- Clean up test resources to avoid costs
- Version test suites alongside infrastructure code

Testing IaC reduces production incidents and builds confidence in infrastructure changes.

### Key Skills
1. **Define infrastructure as code**: Write Terraform configurations, Ansible playbooks, or CloudFormation templates. Use version control for all infrastructure definitions. Create reusable modules and roles. Follow naming conventions and organization standards.
2. **Manage infrastructure lifecycle**: Plan and apply changes safely. Understand state management and locking. Implement proper workspaces or environments. Handle resource dependencies. Plan for disaster recovery with state backups.
3. **Implement configuration management**: Automate server configuration with Ansible or similar tools. Ensure idempotency in configurations. Manage secrets securely. Handle configuration drift detection and remediation.
4. **Version control infrastructure**: Commit IaC to Git repositories. Use branching strategies for infrastructure changes. Implement pull request reviews. Tag releases for rollback capability. Document changes in commit messages.
5. **Test infrastructure code**: Write unit tests for modules. Perform static analysis and security scanning. Execute integration tests in isolated environments. Validate compliance with organizational policies. Automate testing in CI/CD pipelines.

## Projects

### Beginner Level

#### Project 1: Multi-Environment Infrastructure with Terraform
**Objective**: Create reusable Terraform modules for different environments
- VPC and networking setup
- EC2/VM instances with auto-scaling
- Load balancers
- Database instances
- Environment-specific variables (dev, staging, prod)
- Remote state management

**Learning Outcomes**: Terraform basics, modules, workspaces, state management

#### Project 2: Server Configuration with Ansible
**Objective**: Automate server setup and configuration
- Web server installation and configuration
- SSL certificate automation
- User and permission management
- Application deployment
- Service management
- Scheduled tasks (cron jobs)

**Learning Outcomes**: Ansible playbooks, roles, idempotency, automation

#### Project 3: Infrastructure Compliance Scanner
**Objective**: Build a tool to validate infrastructure compliance
- Scan Terraform code for security issues
- Check for compliance with company policies
- Validate resource naming conventions
- Cost estimation
- Generate compliance reports

**Learning Outcomes**: Policy as code, compliance, security scanning

### Intermediate Level

#### Project 4: Multi-Cloud Infrastructure Abstraction
**Objective**: Create a unified IaC layer for multiple clouds
- Terraform modules for AWS, Azure, GCP
- Consistent interface across providers
- Cloud-agnostic resource definitions
- Automated cloud selection based on cost
- Migration tooling between clouds

**Learning Outcomes**: Multi-cloud strategies, abstraction, portability

#### Project 5: GitOps Infrastructure Pipeline
**Objective**: Implement GitOps for infrastructure management
- Git as source of truth
- Automated plan on pull request
- Automated apply on merge
- Drift detection and remediation
- Infrastructure testing in CI
- Approval workflows

**Learning Outcomes**: GitOps, CI/CD for infrastructure, automation

#### Project 6: Dynamic Inventory and Configuration System
**Objective**: Build a dynamic configuration management system
- Service discovery integration
- Dynamic Ansible inventory from cloud APIs
- Auto-scaling configuration updates
- Template-based configuration generation
- Secrets management integration
- Configuration drift detection

**Learning Outcomes**: Dynamic configuration, service discovery, automation

### Advanced Level

#### Project 7: Self-Service Infrastructure Platform
**Objective**: Create an internal infrastructure platform
- Web UI for infrastructure requests
- Template catalog (databases, clusters, networks)
- Automated provisioning workflow
- RBAC and approval chains
- Cost tracking and quotas
- Automated decommissioning

**Learning Outcomes**: Platform engineering, self-service, governance

#### Project 8: Infrastructure Testing Framework
**Objective**: Implement comprehensive infrastructure testing
- Unit tests for Terraform modules (Terratest)
- Integration tests for provisioned resources
- Security compliance tests (Checkov, tfsec)
- Performance tests
- Chaos engineering for infrastructure
- Automated test reporting

**Learning Outcomes**: Testing strategies, quality assurance, reliability

#### Project 9: Immutable Infrastructure Platform
**Objective**: Build a system for immutable infrastructure
- Automated image building (Packer)
- Blue-green infrastructure deployments
- Canary infrastructure releases
- Automated rollback mechanisms
- Zero-downtime migrations
- Infrastructure versioning

**Learning Outcomes**: Immutable infrastructure, deployment strategies, reliability

## Resources

### Documentation
- [Terraform Documentation](https://www.terraform.io/docs)
- [Ansible Documentation](https://docs.ansible.com/)
- [Pulumi Documentation](https://www.pulumi.com/docs/)

### Books
- "Terraform: Up & Running" by Yevgeniy Brikman
- "Ansible for DevOps" by Jeff Geerling
- "Infrastructure as Code" by Kief Morris

### Tools
- Terraform
- Ansible
- Pulumi
- CloudFormation
- Terragrunt
- Checkov
- tfsec
- Terratest

### Best Practices
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

### Certifications
- HashiCorp Certified: Terraform Associate
- Red Hat Certified Specialist in Ansible Automation
