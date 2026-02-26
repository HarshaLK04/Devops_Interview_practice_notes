# Terraform Interview Notes - Complete Guide
## From Basics to Advanced

---

## Table of Contents
1. [Fundamentals](#fundamentals)
2. [Core Concepts](#core-concepts)
3. [Configuration Syntax](#configuration-syntax)
4. [State Management](#state-management)
5. [Modules](#modules)
6. [Providers](#providers)
7. [Variables & Outputs](#variables--outputs)
8. [Functions & Expressions](#functions--expressions)
9. [Provisioners](#provisioners)
10. [Workspaces](#workspaces)
11. [Backend Configuration](#backend-configuration)
12. [Best Practices](#best-practices)
13. [Advanced Topics](#advanced-topics)
14. [Common Interview Questions](#common-interview-questions)

---

## Fundamentals

### What is Terraform?
- **Infrastructure as Code (IaC)** tool developed by HashiCorp
- Declarative configuration language (HCL - HashiCorp Configuration Language)
- Enables infrastructure provisioning across multiple cloud providers
- Open-source with enterprise version available

### Key Features
- **Multi-cloud support**: AWS, Azure, GCP, Kubernetes, etc.
- **Execution plans**: Preview changes before applying
- **Resource graph**: Understands dependencies and parallelizes operations
- **State management**: Tracks real-world infrastructure
- **Idempotent**: Same config produces same result

### Terraform vs Other IaC Tools

| Feature | Terraform | CloudFormation | Ansible | Pulumi |
|---------|-----------|----------------|---------|---------|
| Cloud Support | Multi-cloud | AWS only | Multi-cloud | Multi-cloud |
| Language | HCL | JSON/YAML | YAML | Programming languages |
| State | Yes | AWS managed | No | Yes |
| Approach | Declarative | Declarative | Procedural | Declarative |

---

## Core Concepts

### Terraform Workflow
```
1. Write → 2. Plan → 3. Apply → 4. Destroy
```

**Commands:**
- `terraform init` - Initialize working directory
- `terraform plan` - Preview changes
- `terraform apply` - Execute changes
- `terraform destroy` - Remove infrastructure
- `terraform validate` - Validate configuration syntax
- `terraform fmt` - Format code to canonical style
- `terraform show` - Show current state
- `terraform refresh` - Update state with real infrastructure

### Resource Block
Basic building block representing infrastructure components:
```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
  }
}
```

**Syntax**: `resource "<provider>_<resource_type>" "<local_name>"`

### Data Sources
Read-only information from providers:
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
  
  owners = ["099720109477"] # Canonical
}
```

### Provider
Plugin that interacts with cloud platforms:
```hcl
provider "aws" {
  region  = "us-east-1"
  profile = "default"
}
```

---

## Configuration Syntax

### HCL Basics
```hcl
# Single line comment

/* 
Multi-line
comment
*/

# Variable declaration
variable "instance_type" {
  type        = string
  default     = "t2.micro"
  description = "EC2 instance type"
}

# Local values
locals {
  common_tags = {
    Environment = "production"
    ManagedBy   = "Terraform"
  }
}

# Output
output "instance_ip" {
  value = aws_instance.web.public_ip
}
```

### Data Types
- **Primitive**: `string`, `number`, `bool`
- **Complex**: `list`, `map`, `set`, `object`, `tuple`

```hcl
variable "string_var" {
  type    = string
  default = "hello"
}

variable "number_var" {
  type    = number
  default = 42
}

variable "bool_var" {
  type    = bool
  default = true
}

variable "list_var" {
  type    = list(string)
  default = ["item1", "item2", "item3"]
}

variable "map_var" {
  type = map(string)
  default = {
    key1 = "value1"
    key2 = "value2"
  }
}

variable "object_var" {
  type = object({
    name = string
    age  = number
  })
  default = {
    name = "John"
    age  = 30
  }
}
```

### Resource Meta-Arguments
- **depends_on**: Explicit dependencies
- **count**: Create multiple instances
- **for_each**: Iterate over collections
- **provider**: Select provider configuration
- **lifecycle**: Manage resource lifecycle

```hcl
# depends_on
resource "aws_instance" "app" {
  # ...
  depends_on = [aws_security_group.allow_tls]
}

# count
resource "aws_instance" "server" {
  count         = 3
  ami           = "ami-12345678"
  instance_type = "t2.micro"
  
  tags = {
    Name = "server-${count.index}"
  }
}

# for_each
resource "aws_iam_user" "users" {
  for_each = toset(["user1", "user2", "user3"])
  name     = each.key
}

# lifecycle
resource "aws_instance" "example" {
  # ...
  
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = false
    ignore_changes        = [tags]
  }
}
```

---

## State Management

### What is Terraform State?
- JSON file (`terraform.tfstate`) tracking infrastructure
- Maps configuration to real-world resources
- Stores metadata and resource attributes
- **CRITICAL**: Contains sensitive information

### State File Structure
```json
{
  "version": 4,
  "terraform_version": "1.5.0",
  "serial": 1,
  "lineage": "unique-id",
  "resources": [
    {
      "mode": "managed",
      "type": "aws_instance",
      "name": "web",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [...]
    }
  ]
}
```

### State Commands
```bash
# List resources in state
terraform state list

# Show specific resource
terraform state show aws_instance.web

# Move resource (rename)
terraform state mv aws_instance.old aws_instance.new

# Remove resource from state (doesn't destroy)
terraform state rm aws_instance.web

# Pull remote state to local
terraform state pull

# Push local state to remote
terraform state push

# Replace provider address
terraform state replace-provider old new
```

### Remote State
Store state file remotely for collaboration:

**S3 Backend:**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}
```

**Benefits:**
- Team collaboration
- State locking (prevents concurrent modifications)
- Versioning and backup
- Security (encryption, access control)

### State Locking
Prevents concurrent operations:
- Enabled automatically with DynamoDB (AWS)
- Azure Blob Storage supports native locking
- Prevents state corruption

```bash
# Force unlock (use carefully!)
terraform force-unlock <LOCK_ID>
```

### State File Security
- Never commit to version control
- Use `.gitignore`:
  ```
  *.tfstate
  *.tfstate.backup
  .terraform/
  ```
- Enable encryption at rest
- Use IAM policies for access control
- Consider state file versioning

---

## Modules

### What are Modules?
Containers for multiple resources used together. Every Terraform configuration has at least one module (root module).

### Module Structure
```
module/
├── main.tf       # Primary resources
├── variables.tf  # Input variables
├── outputs.tf    # Output values
├── README.md     # Documentation
└── versions.tf   # Provider requirements
```

### Creating a Module
**variables.tf:**
```hcl
variable "instance_type" {
  type        = string
  description = "EC2 instance type"
}

variable "ami_id" {
  type = string
}
```

**main.tf:**
```hcl
resource "aws_instance" "server" {
  ami           = var.ami_id
  instance_type = var.instance_type
}
```

**outputs.tf:**
```hcl
output "instance_id" {
  value = aws_instance.server.id
}

output "public_ip" {
  value = aws_instance.server.public_ip
}
```

### Using a Module
```hcl
module "web_server" {
  source = "./modules/ec2-instance"
  
  instance_type = "t2.micro"
  ami_id        = "ami-12345678"
}

# Access module outputs
output "web_server_ip" {
  value = module.web_server.public_ip
}
```

### Module Sources
```hcl
# Local path
module "local" {
  source = "./modules/vpc"
}

# Terraform Registry
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.0.0"
}

# GitHub
module "github" {
  source = "github.com/user/repo//modules/vpc?ref=v1.0.0"
}

# Git
module "git" {
  source = "git::https://example.com/vpc.git?ref=v1.2.0"
}

# S3
module "s3" {
  source = "s3::https://s3.amazonaws.com/bucket/module.zip"
}
```

### Module Best Practices
- Single responsibility principle
- Well-defined inputs and outputs
- Comprehensive documentation
- Version pinning
- Example usage in README
- Avoid hardcoded values

---

## Providers

### Provider Configuration
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
  
  required_version = ">= 1.5.0"
}

provider "aws" {
  region = "us-east-1"
  
  default_tags {
    tags = {
      Environment = "production"
      ManagedBy   = "Terraform"
    }
  }
}
```

### Multiple Provider Instances (Aliases)
```hcl
provider "aws" {
  region = "us-east-1"
  alias  = "east"
}

provider "aws" {
  region = "us-west-2"
  alias  = "west"
}

resource "aws_instance" "east_server" {
  provider = aws.east
  # ...
}

resource "aws_instance" "west_server" {
  provider = aws.west
  # ...
}
```

### Provider Version Constraints
```hcl
version = "1.2.3"    # Exact version
version = ">= 1.2"   # Greater than or equal
version = "<= 1.2"   # Less than or equal
version = "~> 1.2"   # Any version in 1.x range (>= 1.2, < 2.0)
version = "~> 1.2.3" # Patch updates only (>= 1.2.3, < 1.3.0)
```

### Provider Authentication

**AWS:**
```hcl
provider "aws" {
  region     = "us-east-1"
  access_key = var.aws_access_key  # Not recommended
  secret_key = var.aws_secret_key  # Not recommended
  
  # Better: Use AWS profile
  profile = "default"
  
  # Or assume role
  assume_role {
    role_arn = "arn:aws:iam::123456789012:role/TerraformRole"
  }
}
```

**Azure:**
```hcl
provider "azurerm" {
  features {}
  
  subscription_id = var.subscription_id
  tenant_id       = var.tenant_id
  # Use service principal or managed identity
}
```

---

## Variables & Outputs

### Input Variables

**Variable Declaration:**
```hcl
variable "name" {
  type        = string
  description = "Description of the variable"
  default     = "default-value"
  sensitive   = false
  
  validation {
    condition     = length(var.name) > 3
    error_message = "Name must be more than 3 characters."
  }
}
```

**Variable Types:**
```hcl
variable "string_var" {
  type = string
}

variable "number_var" {
  type = number
}

variable "bool_var" {
  type = bool
}

variable "list_var" {
  type = list(string)
}

variable "set_var" {
  type = set(string)
}

variable "map_var" {
  type = map(string)
}

variable "object_var" {
  type = object({
    name = string
    age  = number
  })
}

variable "tuple_var" {
  type = tuple([string, number, bool])
}

variable "any_var" {
  type = any
}
```

**Assigning Values:**

1. **Command line:**
   ```bash
   terraform apply -var="instance_type=t2.large"
   ```

2. **Variable file (terraform.tfvars):**
   ```hcl
   instance_type = "t2.large"
   region        = "us-west-2"
   ```

3. **Custom variable file:**
   ```bash
   terraform apply -var-file="production.tfvars"
   ```

4. **Environment variables:**
   ```bash
   export TF_VAR_instance_type="t2.large"
   ```

5. **Interactive prompt** (if no value provided)

**Variable Precedence (lowest to highest):**
1. Environment variables
2. terraform.tfvars
3. *.auto.tfvars (alphabetical order)
4. -var and -var-file (command line)

### Output Values

```hcl
output "instance_ip" {
  value       = aws_instance.web.public_ip
  description = "Public IP of the web server"
  sensitive   = false
}

output "db_connection" {
  value     = aws_db_instance.main.connection_string
  sensitive = true  # Won't show in console output
}

# Output with depends_on
output "vpc_info" {
  value      = data.aws_vpc.main.id
  depends_on = [aws_vpc.main]
}
```

**Accessing Outputs:**
```bash
# Show all outputs
terraform output

# Show specific output
terraform output instance_ip

# JSON format
terraform output -json

# Use in another configuration
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = "terraform-state"
    key    = "vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.vpc.outputs.subnet_id
}
```

### Local Values
```hcl
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "Terraform"
  }
  
  # Computed values
  instance_name = "${var.project_name}-${var.environment}-server"
  
  # Conditional values
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
}

resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = local.instance_type
  
  tags = merge(
    local.common_tags,
    {
      Name = local.instance_name
    }
  )
}
```

---

## Functions & Expressions

### String Functions
```hcl
# join - Join list elements with separator
join(", ", ["a", "b", "c"])  # "a, b, c"

# split - Split string into list
split(",", "a,b,c")  # ["a", "b", "c"]

# upper/lower - Change case
upper("hello")  # "HELLO"
lower("WORLD")  # "world"

# format - String formatting
format("Hello, %s!", "World")  # "Hello, World!"

# replace - Replace substring
replace("hello world", "world", "terraform")  # "hello terraform"

# substr - Extract substring
substr("hello", 0, 3)  # "hel"

# trimspace - Remove whitespace
trimspace("  hello  ")  # "hello"
```

### Collection Functions
```hcl
# length - Get collection size
length([1, 2, 3])  # 3

# concat - Merge lists
concat([1, 2], [3, 4])  # [1, 2, 3, 4]

# merge - Merge maps
merge({a = 1}, {b = 2})  # {a = 1, b = 2}

# contains - Check membership
contains([1, 2, 3], 2)  # true

# distinct - Remove duplicates
distinct([1, 2, 2, 3])  # [1, 2, 3]

# flatten - Flatten nested lists
flatten([[1, 2], [3, 4]])  # [1, 2, 3, 4]

# keys - Get map keys
keys({a = 1, b = 2})  # ["a", "b"]

# values - Get map values
values({a = 1, b = 2})  # [1, 2]

# lookup - Get map value with default
lookup({a = 1}, "b", 0)  # 0

# element - Get list element (wraps around)
element([1, 2, 3], 5)  # 3 (index wraps)

# slice - Extract list slice
slice([1, 2, 3, 4], 1, 3)  # [2, 3]
```

### Numeric Functions
```hcl
# min/max - Minimum/maximum
min(1, 2, 3)  # 1
max(1, 2, 3)  # 3

# ceil/floor - Round up/down
ceil(1.2)   # 2
floor(1.9)  # 1

# abs - Absolute value
abs(-5)  # 5

# pow - Power
pow(2, 3)  # 8
```

### Type Conversion Functions
```hcl
# tostring/tonumber/tobool
tostring(123)      # "123"
tonumber("123")    # 123
tobool("true")     # true

# tolist/toset/tomap
tolist([1, 2, 3])  # [1, 2, 3]
toset([1, 2, 2])   # [1, 2]
tomap({a = 1})     # {a = 1}
```

### Date/Time Functions
```hcl
# timestamp - Current UTC time
timestamp()  # "2024-01-15T12:30:45Z"

# formatdate - Format timestamp
formatdate("DD MMM YYYY hh:mm ZZZ", timestamp())
# "15 Jan 2024 12:30 UTC"

# timeadd - Add duration
timeadd(timestamp(), "24h")  # Add 24 hours
```

### Encoding Functions
```hcl
# jsonencode/jsondecode - JSON encoding
jsonencode({a = 1, b = 2})  # '{"a":1,"b":2}'
jsondecode('{"a":1}')        # {a = 1}

# yamlencode/yamldecode - YAML encoding
yamlencode({a = 1, b = 2})

# base64encode/base64decode
base64encode("hello")  # "aGVsbG8="
base64decode("aGVsbG8=")  # "hello"
```

### File Functions
```hcl
# file - Read file content
file("${path.module}/config.txt")

# fileexists - Check file existence
fileexists("${path.module}/config.txt")

# templatefile - Render template
templatefile("${path.module}/template.tpl", {
  name = "John"
  age  = 30
})
```

### Conditional Expressions
```hcl
# Ternary operator
condition ? true_val : false_val

# Example
instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"

# Complex conditions
instance_count = (
  var.environment == "prod" ? 3 :
  var.environment == "staging" ? 2 :
  1
)
```

### For Expressions
```hcl
# List comprehension
[for s in var.list : upper(s)]

# Map comprehension
{for k, v in var.map : k => upper(v)}

# Filtering
[for s in var.list : s if length(s) > 5]

# Index-based
[for i, v in var.list : "${i}: ${v}"]

# Complex transformation
{
  for user in var.users :
  user.id => {
    name  = user.name
    email = user.email
  }
}
```

### Splat Expressions
```hcl
# Get attribute from all resources
aws_instance.servers[*].id

# Equivalent to
[for s in aws_instance.servers : s.id]

# With data sources
data.aws_subnet.all[*].id
```

### Dynamic Blocks
```hcl
resource "aws_security_group" "example" {
  name = "example"
  
  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

---

## Provisioners

### Types of Provisioners

**1. local-exec** - Runs command on machine running Terraform
```hcl
resource "aws_instance" "web" {
  # ...
  
  provisioner "local-exec" {
    command = "echo ${self.private_ip} >> private_ips.txt"
  }
}
```

**2. remote-exec** - Runs commands on remote resource
```hcl
resource "aws_instance" "web" {
  # ...
  
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
  
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
      "sudo systemctl start nginx"
    ]
  }
  
  # Or use script
  provisioner "remote-exec" {
    script = "path/to/script.sh"
  }
}
```

**3. file** - Copy files to remote resource
```hcl
resource "aws_instance" "web" {
  # ...
  
  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    host        = self.public_ip
  }
  
  provisioner "file" {
    source      = "app.conf"
    destination = "/etc/app/app.conf"
  }
  
  # Upload directory
  provisioner "file" {
    source      = "configs/"
    destination = "/tmp/configs"
  }
}
```

### Provisioner Timing
```hcl
resource "aws_instance" "web" {
  # ...
  
  # Run on creation (default)
  provisioner "local-exec" {
    when    = create
    command = "echo 'Instance created'"
  }
  
  # Run on destruction
  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Instance destroyed'"
  }
}
```

### Provisioner Failure Handling
```hcl
resource "aws_instance" "web" {
  # ...
  
  provisioner "remote-exec" {
    on_failure = continue  # or: fail (default)
    inline = [
      "sudo apt-get update"
    ]
  }
}
```

### Why Avoid Provisioners?
- **Not idempotent**: Run only once
- **Hidden dependencies**: Not visible in resource graph
- **Error-prone**: Network issues, timing problems
- **Better alternatives**: 
  - Cloud-init / user_data
  - Configuration management (Ansible, Chef, Puppet)
  - Custom AMIs / container images
  - Immutable infrastructure

**Example using user_data (preferred):**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
  
  user_data = <<-EOF
              #!/bin/bash
              sudo apt-get update
              sudo apt-get install -y nginx
              sudo systemctl start nginx
              EOF
}
```

---

## Workspaces

### What are Workspaces?
Multiple named state files within same configuration, useful for managing different environments.

### Workspace Commands
```bash
# List workspaces
terraform workspace list

# Show current workspace
terraform workspace show

# Create new workspace
terraform workspace new dev

# Switch workspace
terraform workspace select prod

# Delete workspace
terraform workspace delete staging
```

### Using Workspaces
```hcl
resource "aws_instance" "server" {
  ami           = var.ami_id
  instance_type = terraform.workspace == "prod" ? "t3.large" : "t3.micro"
  
  tags = {
    Name        = "server-${terraform.workspace}"
    Environment = terraform.workspace
  }
}

# Different backend per workspace
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "env:/${terraform.workspace}/terraform.tfstate"
    region = "us-east-1"
  }
}
```

### Workspace Best Practices
- **Default workspace**: Cannot be deleted, use for development
- **Naming**: Use consistent naming (dev, staging, prod)
- **Limitations**: 
  - Same backend for all workspaces
  - State isolation not perfect
  - Can lead to drift
- **Better alternative**: Separate directories/repos for environments

---

## Backend Configuration

### Available Backends
- **local** - Default, stores state locally
- **s3** - AWS S3
- **azurerm** - Azure Storage
- **gcs** - Google Cloud Storage
- **consul** - HashiCorp Consul
- **kubernetes** - Kubernetes Secret
- **remote** - Terraform Cloud/Enterprise

### S3 Backend (Complete Example)
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    kms_key_id     = "arn:aws:kms:us-east-1:123456789012:key/12345678"
    dynamodb_table = "terraform-state-lock"
    
    # Optional: Role assumption
    role_arn       = "arn:aws:iam::123456789012:role/TerraformRole"
  }
}
```

**DynamoDB Table for Locking:**
```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-state-lock"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

### Azure Backend
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "terraform-state-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
```

### GCS Backend
```hcl
terraform {
  backend "gcs" {
    bucket = "tf-state-bucket"
    prefix = "terraform/state"
  }
}
```

### Terraform Cloud Backend
```hcl
terraform {
  cloud {
    organization = "my-org"
    
    workspaces {
      name = "my-workspace"
    }
  }
}
```

### Backend Initialization
```bash
# Initialize backend
terraform init

# Migrate state to new backend
terraform init -migrate-state

# Reconfigure backend
terraform init -reconfigure

# Backend configuration via file
terraform init -backend-config=backend.hcl
```

**backend.hcl:**
```hcl
bucket = "my-terraform-state"
key    = "prod/terraform.tfstate"
region = "us-east-1"
```

### Partial Backend Configuration
```hcl
# main.tf
terraform {
  backend "s3" {}
}
```

```bash
# Provide config at init
terraform init \
  -backend-config="bucket=my-bucket" \
  -backend-config="key=my-key" \
  -backend-config="region=us-east-1"
```

---

## Best Practices

### Code Organization
```
project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   └── backend.tf
│   ├── staging/
│   └── prod/
├── modules/
│   ├── vpc/
│   ├── compute/
│   └── database/
└── global/
    └── iam/
```

### Naming Conventions
```hcl
# Resource naming: <resource_type>_<descriptor>
resource "aws_instance" "web_server" { }
resource "aws_security_group" "web_sg" { }

# Variable naming: lowercase with underscores
variable "instance_type" { }
variable "vpc_cidr_block" { }

# Tag naming: PascalCase
tags = {
  Name        = "WebServer"
  Environment = "Production"
  ManagedBy   = "Terraform"
}
```

### Version Constraints
```hcl
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### State Management Best Practices
1. **Use remote backend** with encryption
2. **Enable state locking** to prevent conflicts
3. **Never commit state files** to version control
4. **Regular backups** of state files
5. **Use workspaces judiciously** (prefer separate configs)
6. **Sensitive data**: Mark outputs as sensitive

### Security Best Practices
```hcl
# 1. Use variables for sensitive data
variable "db_password" {
  type      = string
  sensitive = true
}

# 2. Don't hardcode credentials
# BAD:
provider "aws" {
  access_key = "AKIAIOSFODNN7EXAMPLE"  # Never do this!
  secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
}

# GOOD:
provider "aws" {
  # Use AWS credentials file or IAM roles
  profile = "default"
}

# 3. Encrypt state file
terraform {
  backend "s3" {
    encrypt = true
  }
}

# 4. Use sensitive outputs
output "db_password" {
  value     = random_password.db_password.result
  sensitive = true
}

# 5. Scan for secrets
# Use tools like tfsec, checkov, terrascan
```

### Documentation
```hcl
# Always include comments
# This security group allows HTTP and HTTPS traffic

variable "allowed_cidr_blocks" {
  type        = list(string)
  description = "List of CIDR blocks allowed to access the load balancer"
  default     = ["0.0.0.0/0"]
}

# README.md for modules
# - Purpose
# - Requirements
# - Usage examples
# - Input variables
# - Outputs
# - Dependencies
```

### Testing Strategy
1. **Validation**: `terraform validate`
2. **Formatting**: `terraform fmt -check`
3. **Plan review**: `terraform plan`
4. **Static analysis**: tfsec, checkov, terrascan
5. **Policy enforcement**: Sentinel, OPA
6. **Integration testing**: Terratest, kitchen-terraform

### Change Management
```bash
# 1. Always run plan before apply
terraform plan -out=tfplan

# 2. Review plan output carefully
terraform show tfplan

# 3. Apply using saved plan
terraform apply tfplan

# 4. Use target for specific resources (sparingly)
terraform apply -target=aws_instance.web

# 5. Use refresh=false to avoid surprises
terraform plan -refresh=false
```

### Resource Tagging
```hcl
# Use consistent tagging
locals {
  common_tags = {
    Environment = var.environment
    Project     = var.project_name
    ManagedBy   = "Terraform"
    Owner       = var.owner_email
    CostCenter  = var.cost_center
    Compliance  = var.compliance_level
  }
}

resource "aws_instance" "web" {
  # ...
  tags = merge(
    local.common_tags,
    {
      Name = "web-server"
      Role = "frontend"
    }
  )
}
```

### Dependency Management
```hcl
# Implicit dependencies (preferred)
resource "aws_instance" "web" {
  subnet_id = aws_subnet.main.id  # Implicit dependency
}

# Explicit dependencies (when needed)
resource "aws_instance" "web" {
  # ...
  depends_on = [
    aws_security_group.allow_tls,
    aws_iam_role_policy_attachment.example
  ]
}
```

---

## Advanced Topics

### Import Existing Resources
```bash
# Import existing AWS instance
terraform import aws_instance.web i-1234567890abcdef0

# Import with module
terraform import module.vpc.aws_vpc.main vpc-12345678
```

**Write configuration first:**
```hcl
resource "aws_instance" "web" {
  # Match existing resource attributes
  ami           = "ami-12345678"
  instance_type = "t2.micro"
  # ...
}
```

### Terraform Taint (Deprecated in v0.15.2+)
```bash
# Old way (deprecated)
terraform taint aws_instance.web

# New way (v0.15.2+)
terraform apply -replace="aws_instance.web"
```

### Terraform Graph
```bash
# Generate dependency graph
terraform graph | dot -Tpng > graph.png

# Filter by type
terraform graph -type=plan
terraform graph -type=apply
```

### Debugging
```bash
# Enable detailed logs
export TF_LOG=TRACE
export TF_LOG_PATH=terraform.log

# Log levels: TRACE, DEBUG, INFO, WARN, ERROR

# Provider-specific logs
export TF_LOG_PROVIDER=DEBUG

# Disable logs
unset TF_LOG
```

### Null Resource
Execute provisioners without associated resource:
```hcl
resource "null_resource" "example" {
  triggers = {
    always_run = timestamp()
  }
  
  provisioner "local-exec" {
    command = "echo 'Running provisioner'"
  }
}
```

### Terraform Data
```hcl
# terraform_remote_state
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = "my-terraform-state"
    key    = "vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

# Use outputs from other state
resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.vpc.outputs.subnet_id
}
```

### Sensitive Data Handling
```hcl
# Mark variable as sensitive
variable "db_password" {
  type      = string
  sensitive = true
}

# Mark output as sensitive
output "db_password" {
  value     = var.db_password
  sensitive = true
}

# Mark entire resource attribute as sensitive
resource "aws_db_instance" "main" {
  password = var.db_password
  
  lifecycle {
    ignore_changes = [password]
  }
}
```

### Moved Block (Refactoring)
```hcl
# Rename resource without destroying
moved {
  from = aws_instance.old_name
  to   = aws_instance.new_name
}

# Move to module
moved {
  from = aws_instance.web
  to   = module.web_server.aws_instance.web
}
```

### Custom Validation Rules
```hcl
variable "instance_type" {
  type = string
  
  validation {
    condition = contains([
      "t2.micro",
      "t2.small",
      "t3.micro",
      "t3.small"
    ], var.instance_type)
    error_message = "Instance type must be a valid t2 or t3 type."
  }
}

variable "environment" {
  type = string
  
  validation {
    condition     = can(regex("^(dev|staging|prod)$", var.environment))
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "cidr_block" {
  type = string
  
  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be a valid CIDR block."
  }
}
```

### Preconditions and Postconditions
```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  lifecycle {
    # Check before creating
    precondition {
      condition     = data.aws_ami.main.architecture == "x86_64"
      error_message = "AMI must be x86_64 architecture."
    }
    
    # Check after creating
    postcondition {
      condition     = self.public_dns != ""
      error_message = "Instance must have a public DNS name."
    }
  }
}
```

### Override Files
```hcl
# main.tf
resource "aws_instance" "web" {
  instance_type = "t2.micro"
}

# override.tf (or main_override.tf)
resource "aws_instance" "web" {
  instance_type = "t3.large"  # Overrides the above
}
```

### Terraform Functions - Advanced Usage
```hcl
# Complex transformations
locals {
  # Group resources by attribute
  instances_by_az = {
    for instance in aws_instance.servers :
    instance.availability_zone => instance...
  }
  
  # Flatten nested structures
  all_security_rules = flatten([
    for sg in var.security_groups : [
      for rule in sg.rules : {
        sg_id   = sg.id
        rule_id = rule.id
        port    = rule.port
      }
    ]
  ])
  
  # Conditional inclusion
  enabled_features = [
    for feature, enabled in var.features :
    feature if enabled
  ]
}
```

### Provider Aliases in Modules
```hcl
# In module
terraform {
  required_providers {
    aws = {
      source                = "hashicorp/aws"
      configuration_aliases = [aws.replica]
    }
  }
}

resource "aws_s3_bucket" "primary" {
  bucket = "primary-bucket"
}

resource "aws_s3_bucket" "replica" {
  provider = aws.replica
  bucket   = "replica-bucket"
}

# Using module
provider "aws" {
  alias  = "us_east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "us_west"
  region = "us-west-2"
}

module "s3_replication" {
  source = "./modules/s3-replication"
  
  providers = {
    aws         = aws.us_east
    aws.replica = aws.us_west
  }
}
```

### Terraform Cloud/Enterprise Features
- **Remote execution**: Run Terraform in the cloud
- **VCS integration**: Automatic runs on commit
- **Policy as code**: Sentinel/OPA policies
- **Private module registry**: Share modules securely
- **Team management**: RBAC and audit logs
- **Cost estimation**: Preview infrastructure costs
- **Run triggers**: Chain workspace runs

---

## Common Interview Questions

### Basic Level

**Q1: What is Terraform and why use it?**
A: Terraform is an IaC tool that allows you to define infrastructure in declarative configuration files. Benefits include version control, automation, consistency, multi-cloud support, and preview of changes.

**Q2: What is terraform init?**
A: Initializes a Terraform working directory by:
- Downloading provider plugins
- Initializing backend
- Creating .terraform directory
- Installing modules

**Q3: Difference between terraform plan and terraform apply?**
A: `plan` generates execution plan showing changes without making them. `apply` executes the plan and makes actual changes to infrastructure.

**Q4: What is terraform state?**
A: JSON file tracking real-world resource mappings, metadata, and resource attributes. Critical for Terraform operations and contains sensitive data.

**Q5: What are providers in Terraform?**
A: Plugins that enable Terraform to interact with cloud platforms, SaaS providers, and APIs. Examples: AWS, Azure, GCP, Kubernetes.

### Intermediate Level

**Q6: How does Terraform handle dependencies?**
A: Two ways:
- **Implicit**: Automatic via resource attribute references
- **Explicit**: Using `depends_on` meta-argument

**Q7: What is the difference between count and for_each?**
A:
- **count**: Creates numbered instances (0, 1, 2...). Limitations with list changes.
- **for_each**: Creates instances identified by unique keys (map/set). Better for managing individual resources.

**Q8: How to manage different environments (dev/staging/prod)?**
A: Multiple approaches:
1. Separate directories with different tfvars
2. Workspaces (same config, different state)
3. Branches in version control
4. Separate repos per environment

**Q9: What is remote state and why use it?**
A: Storing state file remotely (S3, Azure Blob) for:
- Team collaboration
- State locking
- Versioning and backup
- Security (encryption, access control)

**Q10: What are modules and when to use them?**
A: Reusable containers of Terraform configuration. Use for:
- Code reusability
- Abstraction
- Consistency
- Organization
- Encapsulation

### Advanced Level

**Q11: How to import existing infrastructure into Terraform?**
A: Use `terraform import` command:
1. Write resource configuration matching existing infrastructure
2. Run `terraform import <resource_address> <resource_id>`
3. Verify with `terraform plan` (should show no changes)

**Q12: How does state locking work?**
A: Prevents concurrent state modifications:
- Automatic with backends supporting locking (DynamoDB for S3)
- Locks acquired during write operations
- Released after completion
- Can force-unlock if needed

**Q13: What is taint and when to use it?**
A: Marking resource for recreation on next apply. Now use `terraform apply -replace`. Used when:
- Resource is corrupted
- Configuration outside Terraform changed it
- Force recreation needed

**Q14: How to handle secrets in Terraform?**
A:
- Mark variables/outputs as sensitive
- Use external secret managers (Vault, AWS Secrets Manager)
- Environment variables
- Encrypt state file
- Never commit secrets to VCS

**Q15: What are provisioners and why avoid them?**
A: Execute scripts on local/remote machines. Avoid because:
- Not idempotent
- Error-prone
- Hidden dependencies
- Better alternatives: cloud-init, AMIs, config management tools

**Q16: Explain terraform refresh**
A: Updates state file with real-world infrastructure without modifying resources. Now implicit in plan/apply. Use `-refresh=false` to skip.

**Q17: What is the difference between local and remote exec provisioners?**
A:
- **local-exec**: Runs on machine executing Terraform
- **remote-exec**: Runs on the remote resource being created

**Q18: How to handle sensitive outputs?**
A:
```hcl
output "password" {
  value     = random_password.db.result
  sensitive = true  # Hidden from console
}
```

**Q19: Explain terraform graph**
A: Generates visual representation of resource dependency graph in DOT format. Useful for understanding relationships and debugging.

**Q20: What is lifecycle meta-argument?**
A: Controls resource lifecycle behavior:
- `create_before_destroy`: Create new before destroying old
- `prevent_destroy`: Prevent accidental destruction
- `ignore_changes`: Ignore specified attribute changes
- `replace_triggered_by`: Force replacement when specified resources change

### Scenario-Based Questions

**Q21: You applied infrastructure but need to roll back. How?**
A:
1. Revert code changes in VCS
2. Run `terraform plan` to verify
3. Run `terraform apply`
4. If state corrupted, restore from backup
5. Alternative: Use version control on state files

**Q22: Multiple team members are getting state lock errors. What to do?**
A:
1. Verify no one is actually running Terraform
2. Check for crashed processes
3. Use `terraform force-unlock <LOCK_ID>` carefully
4. Implement proper state locking with DynamoDB/similar
5. Consider using Terraform Cloud for better collaboration

**Q23: How to organize Terraform code for large projects?**
A:
```
project/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
├── modules/
│   ├── networking/
│   ├── compute/
│   └── database/
└── global/
    └── iam/
```

**Q24: You need to update 100 resources but only want to change 1. How?**
A: Use `terraform apply -target=resource.name`
Warning: Can lead to drift, use sparingly.

**Q25: State file is corrupted. Recovery steps?**
A:
1. Restore from backup (S3 versioning, etc.)
2. If no backup, manually reconstruct:
   - Create empty state
   - Import resources one by one
   - Verify with plan
3. Prevention: Always use remote backend with versioning

### Best Practices Questions

**Q26: Security best practices for Terraform?**
A:
- Never commit state files or secrets
- Use remote backend with encryption
- Mark sensitive variables/outputs
- Use IAM roles instead of access keys
- Enable state locking
- Regular security scans (tfsec, checkov)
- Principle of least privilege
- Audit logs and access control

**Q27: How to test Terraform code?**
A:
1. **Validation**: `terraform validate`
2. **Formatting**: `terraform fmt`
3. **Static analysis**: tfsec, checkov, terrascan
4. **Policy checks**: Sentinel, OPA
5. **Unit tests**: Terratest
6. **Plan review**: Manual inspection
7. **Integration tests**: Deploy to test environment

**Q28: How to handle Terraform version upgrades?**
A:
1. Review CHANGELOG
2. Test in non-prod environment
3. Update version constraints
4. Run `terraform init -upgrade`
5. Test with `terraform plan`
6. Check for deprecations
7. Update state file format if needed

**Q29: What's the difference between terraform.tfvars and variables.tf?**
A:
- **variables.tf**: Declares variables (type, default, description)
- **terraform.tfvars**: Assigns values to variables
- **.tfvars** files are for values, **.tf** files are for declarations

**Q30: How to manage Terraform state for multiple environments?**
A:
1. **Separate state files**: Different backend keys
2. **Workspaces**: Same config, different state
3. **Separate directories**: Complete isolation
4. **Different repos**: Maximum separation
Choose based on blast radius and team structure.

---

## Troubleshooting Guide

### Common Errors and Solutions

**1. State Lock Error**
```
Error: Error acquiring the state lock
```
**Solution:**
```bash
terraform force-unlock <LOCK_ID>
```

**2. Provider Plugin Not Found**
```
Error: provider registry.terraform.io/hashicorp/aws: no suitable version installed
```
**Solution:**
```bash
terraform init -upgrade
```

**3. Cycle Error**
```
Error: Cycle: resource1 -> resource2 -> resource1
```
**Solution:** Remove circular dependencies using depends_on or restructure resources.

**4. Authentication Errors**
```
Error: Error configuring the backend "s3": No valid credential sources found
```
**Solution:** Configure AWS credentials properly:
```bash
aws configure
# or
export AWS_PROFILE=myprofile
```

**5. State Divergence**
```
Error: Refreshing state... remote state differs from local state
```
**Solution:**
```bash
terraform refresh
terraform plan
```

**6. Resource Already Exists**
```
Error: resource already exists
```
**Solution:** Import the existing resource:
```bash
terraform import <resource_address> <resource_id>
```

---

## Performance Optimization

### Parallelism
```bash
# Default parallelism is 10
terraform apply -parallelism=20

# Reduce for API rate limits
terraform apply -parallelism=1
```

### Reduce Plan Time
```bash
# Skip refresh to speed up plan
terraform plan -refresh=false

# Target specific resources
terraform plan -target=module.vpc
```

### State File Optimization
- Keep state files small
- Split large infrastructures into multiple states
- Use data sources instead of remote state when possible
- Regular state file cleanup

### Provider Caching
```hcl
# Share provider plugins across projects
plugin_cache_dir = "$HOME/.terraform.d/plugin-cache"
```

---

## Terraform Cheat Sheet

### Essential Commands
```bash
# Initialization & Planning
terraform init                    # Initialize directory
terraform init -upgrade           # Upgrade providers
terraform validate                # Validate configuration
terraform fmt                     # Format code
terraform fmt -recursive          # Format all files
terraform plan                    # Generate execution plan
terraform plan -out=tfplan        # Save plan to file

# Applying Changes
terraform apply                   # Apply changes
terraform apply tfplan            # Apply saved plan
terraform apply -auto-approve     # Skip confirmation
terraform apply -target=resource  # Apply specific resource

# Destruction
terraform destroy                 # Destroy all resources
terraform destroy -target=resource # Destroy specific resource

# State Management
terraform state list              # List resources in state
terraform state show resource     # Show resource details
terraform state mv source dest    # Rename resource
terraform state rm resource       # Remove from state
terraform state pull              # Download remote state
terraform state push              # Upload local state

# Workspace Management
terraform workspace list          # List workspaces
terraform workspace new name      # Create workspace
terraform workspace select name   # Switch workspace
terraform workspace delete name   # Delete workspace

# Outputs & Inspection
terraform output                  # Show all outputs
terraform output name             # Show specific output
terraform show                    # Show current state
terraform graph                   # Generate dependency graph

# Importing & Debugging
terraform import addr id          # Import existing resource
terraform refresh                 # Update state
terraform force-unlock id         # Unlock state
```

### Common Patterns
```hcl
# Conditional Resource
resource "aws_instance" "web" {
  count = var.create_instance ? 1 : 0
  # ...
}

# Dynamic Tags
tags = merge(
  local.common_tags,
  {
    Name = "specific-name"
  }
)

# Complex Variable Validation
validation {
  condition = (
    var.environment == "dev" ||
    var.environment == "staging" ||
    var.environment == "prod"
  )
  error_message = "Environment must be dev, staging, or prod."
}
```

---

## Additional Resources

### Official Documentation
- Terraform Documentation: https://www.terraform.io/docs
- Terraform Registry: https://registry.terraform.io
- HashiCorp Learn: https://learn.hashicorp.com/terraform

### Tools & Utilities
- **tfsec**: Security scanner
- **checkov**: Policy as code
- **terrascan**: Security & compliance
- **terraform-docs**: Generate documentation
- **Terragrunt**: Wrapper for DRY configs
- **Atlantis**: Pull request automation
- **Terraform Cloud**: SaaS offering
- **Infracost**: Cost estimation

### Community Resources
- GitHub: https://github.com/hashicorp/terraform
- Community Forum: https://discuss.hashicorp.com
- Stack Overflow: terraform tag
- Reddit: r/Terraform

---

## Interview Preparation Tips

1. **Understand the basics thoroughly**: State, providers, resources, modules
2. **Practice hands-on**: Set up real infrastructure
3. **Read official documentation**: Stay current with latest features
4. **Study best practices**: Security, organization, testing
5. **Learn debugging**: Common errors and troubleshooting
6. **Know alternatives**: When to use CloudFormation, Ansible, etc.
7. **Stay updated**: Terraform evolves rapidly
8. **Real-world scenarios**: Be prepared for problem-solving questions

### Key Areas to Focus On
- State management and locking
- Module creation and best practices
- Security and secret handling
- Multi-environment management
- Troubleshooting common issues
- Backend configuration
- Provider authentication
- Resource lifecycle management
- Complex expressions and functions

---

## Summary

Terraform is a powerful IaC tool that requires understanding of:
- **Core concepts**: Resources, providers, state, modules
- **Configuration**: HCL syntax, variables, outputs, functions
- **State management**: Remote backends, locking, security
- **Best practices**: Organization, testing, security, documentation
- **Advanced features**: Import, workspaces, custom validation
- **Troubleshooting**: Common errors and their solutions

**Remember**: Practice is key! Set up real infrastructure, break things, fix them, and learn from the experience.

Good luck with your interview! 🚀