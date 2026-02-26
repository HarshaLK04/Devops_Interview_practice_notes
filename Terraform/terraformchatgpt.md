Terraform is one of those deceptively simple tools. At first glance it looks like “just write some .tf files and run apply,” but under the hood it’s doing graph theory, state reconciliation, and dependency resolution like a quiet infrastructure chess engine. Interviewers love Terraform because it exposes whether someone understands infrastructure as a system—or is just copy-pasting from Stack Overflow.

Let’s build your **DevOps interview war-notes**, starting from atoms → universe.

---

# TERRAFORM – COMPLETE INTERVIEW NOTES (BASIC → HARDCORE)

## 1. What is Terraform

![Image](https://cms.cloudoptimo.com/uploads/terraform_eaa800441c.png)

![Image](https://media2.dev.to/cdn-cgi/image/width%3D800%2Cheight%3D%2Cfit%3Dscale-down%2Cgravity%3Dauto%2Cformat%3Dauto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg1ormw9rdlb77vosclve.jpeg)

![Image](https://cdn.prod.website-files.com/644656ba41efb6b601e93ca6/666ca94313bc92617e6eb9fa_AD_4nXe-5_WQu-YNEB3tjjsejMPFliYTzRNjfX5D4sBknnJ9T-25KaQ1UAv3JsxDelee3icN2knxbdR7O6Upx--gqbvpij3hpWqgifxPez8_0ZtHflV45C1BsL3Wzs_tSLjn7WhK9JoiuY6EAd3gAtPfFU3-HaJ-.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/0%2AlPfL5tZkTMamzfh0.jpg)

Terraform is an **Infrastructure as Code (IaC)** tool created by HashiCorp.

It is used to:

• Create infrastructure
• Update infrastructure
• Delete infrastructure

using **code instead of manual work**

Example:

Instead of creating EC2 manually → write code → Terraform creates it.

Supports:

• AWS
• Azure
• GCP
• Kubernetes
• VMware
• almost everything

---

## 2. Why Terraform is used

Interview one-liner:

Terraform is used to automate infrastructure provisioning in a consistent, repeatable, and version-controlled way.

Benefits:

Consistency
Automation
Version control
Reusability
Multi-cloud support

Real DevOps example (your profile relevant):

Create:

• EC2
• VPC
• Load Balancer
• RDS

in single command

---

## 3. Terraform Architecture

Core components:

Terraform Core

Responsible for:

• reading code
• building dependency graph
• execution

Providers

Example:

aws
azure
kubernetes

Provider is plugin which talks to cloud

Example:

Terraform → AWS Provider → AWS API → creates EC2

---

## 4. Terraform Workflow (VERY IMPORTANT INTERVIEW QUESTION)

Commands sequence:

init
plan
apply
destroy

---

### terraform init

Initializes project

downloads providers

creates:

.terraform folder

---

### terraform plan

Shows:

what terraform will create / modify / destroy

NO changes done yet

Safe command

Interview line:

terraform plan is dry run

---

### terraform apply

Actually creates infrastructure

---

### terraform destroy

Deletes infrastructure

---

Interview trick question:

Difference between plan and apply

Answer:

Plan shows changes, apply executes changes

---

## 5. Terraform File Structure

Extension:

.tf

Example:

main.tf

variables.tf

outputs.tf

providers.tf

terraform.tfvars

---

## 6. Basic Terraform Example

AWS EC2

```
provider "aws" {

region = "ap-south-1"

}

resource "aws_instance" "myec2" {

ami = "ami-123456"

instance_type = "t2.micro"

}
```

---

Concepts here:

provider

resource

arguments

---

## 7. Resource Block

Most important block

Syntax:

```
resource "resource_type" "name" {

}

```

Example:

```
resource "aws_instance" "web"

```

aws_instance → type

web → name

---

Interview Question:

What is difference between resource type and name

Answer:

Type is AWS service

Name is local reference

---

## 8. Providers

Provider connects terraform to cloud

Example:

AWS provider

```
provider "aws" {

region = "ap-south-1"

}
```

Without provider Terraform cannot create infra

---

## 9. Terraform State File (VERY VERY IMPORTANT)

File name:

terraform.tfstate

Terraform stores infrastructure info here

It tracks:

what terraform created

Terraform compares:

Code vs State vs Real infra

Interview killer line:

Terraform state is source of truth

---

Example:

Terraform knows:

EC2 exists

its id

its config

---

## 10. Desired State vs Actual State

Terraform works on:

Desired State → Code

Actual State → Real infra

Terraform makes Actual = Desired

This concept is called:

State reconciliation

---

## 11. Variables

Variables make code reusable

Example:

```
variable "instance_type" {

default = "t2.micro"

}
```

Use:

```
instance_type = var.instance_type

```

---

Types:

string

number

bool

list

map

---

Interview question:

Where variables stored

Answer:

variables.tf

terraform.tfvars

environment variables

CLI

---

## 12. terraform.tfvars

Stores values

Example:

```
instance_type = "t2.small"

```

---

## 13. Output Block

Shows output

Example:

```
output "instance_ip" {

value = aws_instance.web.public_ip

}
```

Interview usage:

Used to pass value to other modules

---

## 14. Terraform Modules

Modules are reusable code

Example:

Create EC2 module

use multiple times

Interview definition:

Module is reusable terraform code block

---

Example:

module folder:

main.tf

variables.tf

outputs.tf

use:

```
module "ec2" {

source = "./ec2"

}
```

---

Real DevOps usage:

Company creates modules:

VPC module

EC2 module

RDS module

---

## 15. Terraform Dependency

Terraform automatically detects dependency

Example:

EC2 depends on VPC

Terraform creates VPC first

This is called:

Implicit dependency

---

Explicit dependency:

```
depends_on = [aws_vpc.main]

```

---

## 16. Terraform Lifecycle

Used to control resource behavior

Example:

```
lifecycle {

prevent_destroy = true

}
```

Prevents deletion

Interview important:

prevent_destroy

create_before_destroy

ignore_changes

---

## 17. Backend in Terraform

Backend stores state file

Types:

local backend

remote backend

---

Remote backend example:

S3 backend

```
backend "s3" {

bucket = "terraform-state"

key = "state.tfstate"

region = "ap-south-1"

}
```

---

Interview question:

Why remote backend used

Answer:

Team collaboration

State locking

Security

---

## 18. State Locking

Prevents multiple users from modifying state simultaneously

Example:

Uses DynamoDB in AWS

---

Scenario question:

What happens if two people run terraform apply

Answer:

State locking prevents conflict

---

## 19. Terraform Commands (Important)

init

plan

apply

destroy

validate

fmt

show

refresh

import

taint

untaint

state

---

Interview favorite:

terraform fmt

formats code

terraform validate

checks syntax

---

## 20. terraform import

Imports existing infra

Example:

Existing EC2 → bring under terraform

---

Command:

```bash
terraform import aws_instance.web i-123456
```

---

## 21. terraform taint

Marks resource for recreation

Example:

EC2 corrupted

terraform taint aws_instance.web

---

## 22. Provisioners

Used to execute scripts

Example:

remote-exec

local-exec

Example:

```
provisioner "remote-exec" {

inline = [

"sudo yum install nginx"

]

}
```

Interview reality:

Provisioners not recommended

Use configuration tools like Ansible

---

## 23. Terraform Graph

Terraform creates dependency graph internally

Executes in order

Parallel execution

---

## 24. Workspaces

Used for multiple environments

Example:

dev

test

prod

Command:

terraform workspace new dev

---

Interview scenario:

Same code → different environments

Use workspace

---

## 25. Terraform Functions

Example:

length()

lookup()

element()

join()

file()

---

Example:

```
length(var.list)

```

---

## 26. Count and for_each (VERY IMPORTANT)

Count:

Create multiple resources

```
count = 3

```

Creates 3 EC2

---

for_each:

More powerful

Example:

```
for_each = var.instances

```

Interview question:

Difference between count and for_each

Answer:

count → index based

for_each → key based

for_each preferred

---

## 27. Dynamic Blocks

Used for dynamic configs

Advanced concept

---

## 28. Terraform vs CloudFormation

Interview favorite question

Terraform:

Multi cloud

CloudFormation:

AWS only

Terraform better flexibility

---

## 29. Terraform Best Practices

Use modules

Use remote backend

Use state locking

Use variables

Avoid hardcoding

---

## 30. Terraform Interview Scenario Questions (VERY IMPORTANT)

Scenario 1:

How terraform knows resource already exists

Answer:

Using state file

---

Scenario 2:

How to share state across team

Answer:

S3 backend

---

Scenario 3:

How to manage multiple environments

Answer:

Workspace or separate state

---

Scenario 4:

What happens if state file deleted

Answer:

Terraform loses tracking

infra still exists

---

Scenario 5:

terraform apply failed in middle

Answer:

Terraform updates state for completed resources

---

## 31. Advanced / Hardcore Interview Questions

These separate junior from senior

---

Explain terraform state locking internally

Explain terraform refresh

Explain dependency graph

Explain drift detection

Explain remote backend working

Explain module registry

Explain terraform lifecycle

Explain implicit vs explicit dependency

Explain terraform import

Explain terraform taint

Explain workspace vs separate folder

Explain terraform plan internally

---

## 32. Real DevOps Interview Scenario (VERY COMMON)

Scenario:

Create:

VPC

EC2

ALB

AutoScaling

RDS

using terraform

Expected answer:

Use modules

Use variables

Use remote backend

Use state locking

Use outputs

Use dependency

---

## 33. Interview Ready One-Line Definitions

Terraform

Infrastructure as code tool

Provider

Plugin to communicate with cloud

State file

Tracks infrastructure

Module

Reusable terraform code

Backend

Stores state file

Workspace

Separate environments

---

## 34. Terraform vs Ansible

Terraform

Creates infrastructure

Ansible

Configures infrastructure

Example:

Terraform → create EC2

Ansible → install nginx

---

This combo is exactly what you're doing in VMware Aria + AWX. Terraform builds the machine, Ansible turns it into something useful. Skeleton and muscles.

---

# FINAL MOST IMPORTANT INTERVIEW TOPICS (MEMORIZE THESE)

These are asked in 80% interviews:

Terraform workflow
State file
Backend
Modules
Variables
Count vs for_each
Lifecycle
Import
Provisioners
Dependency
Workspace

---

# If you master this level, you can clear:

DevOps Engineer
SRE
Cloud Engineer
Release Engineer

interviews at companies like:

Infosys
TCS
Wipro
Accenture
Mphasis
Product companies

---

Infrastructure is a strange philosophical object. It exists in three places at once: in your code, in your state file, and in reality. Terraform’s entire job is to reconcile those three truths into one. That’s almost metaphysics disguised as DevOps.

Next level exploration naturally leads into Terraform internals like graph traversal order, state corruption recovery, and zero-downtime deployments using create_before_destroy—those are the moves that signal real mastery.
