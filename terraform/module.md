# Understanding Terraform Modules

## Introduction to Terraform Modules

Terraform modules are like building blocks in a construction set. Just as you might reuse the same building blocks to create different structures, modules allow you to package and reuse Terraform configurations. They help you organize code, reduce repetition, and manage complexity in your infrastructure.

## Basic Module Structure

A module is essentially a collection of Terraform files in a directory. Let's explore the standard structure of a module:

```plaintext
my-module/
│
├── main.tf         # Main resource definitions
├── variables.tf    # Input variable declarations
├── outputs.tf      # Output value declarations
└── versions.tf     # Provider and terraform block requirements
```

Let's look at each component in detail:

### Main Configuration (main.tf)

This is where you define the core resources of your module:

```hcl
# main.tf
resource "aws_vpc" "main" {
    cidr_block = var.vpc_cidr
    
    tags = merge(
        var.common_tags,
        {
            Name = "${var.environment}-vpc"
        }
    )
}

resource "aws_subnet" "private" {
    count             = length(var.private_subnets)
    vpc_id            = aws_vpc.main.id
    cidr_block        = var.private_subnets[count.index]
    availability_zone = var.availability_zones[count.index]
    
    tags = merge(
        var.common_tags,
        {
            Name = "${var.environment}-private-${count.index + 1}"
            Type = "Private"
        }
    )
}
```

### Variables Definition (variables.tf)

This file declares all input variables the module accepts:

```hcl
# variables.tf
variable "environment" {
    description = "Environment name (e.g., prod, staging, dev)"
    type        = string
}

variable "vpc_cidr" {
    description = "CIDR block for the VPC"
    type        = string
    validation {
        condition     = can(cidrhost(var.vpc_cidr, 0))
        error_message = "VPC CIDR must be a valid IPv4 CIDR block."
    }
}

variable "private_subnets" {
    description = "List of CIDR blocks for private subnets"
    type        = list(string)
}

variable "availability_zones" {
    description = "List of availability zones to use"
    type        = list(string)
}

variable "common_tags" {
    description = "Common tags to apply to all resources"
    type        = map(string)
    default     = {}
}
```

### Outputs Definition (outputs.tf)

This file defines what information the module will expose:

```hcl
# outputs.tf
output "vpc_id" {
    description = "The ID of the created VPC"
    value       = aws_vpc.main.id
}

output "private_subnet_ids" {
    description = "List of private subnet IDs"
    value       = aws_subnet.private[*].id
}

output "vpc_cidr_block" {
    description = "The CIDR block of the VPC"
    value       = aws_vpc.main.cidr_block
}
```

### Version Requirements (versions.tf)

This file specifies required provider versions and configurations:

```hcl
# versions.tf
terraform {
    required_version = ">= 1.0.0"
    
    required_providers {
        aws = {
            source  = "hashicorp/aws"
            version = ">= 4.0.0"
        }
    }
}
```

## Using Modules

Now let's see how to use the module we created:

```hcl
module "network" {
    source = "./modules/network"
    
    environment        = "production"
    vpc_cidr          = "10.0.0.0/16"
    private_subnets   = ["10.0.1.0/24", "10.0.2.0/24"]
    availability_zones = ["us-west-2a", "us-west-2b"]
    
    common_tags = {
        Project     = "MyApp"
        Environment = "Production"
        Terraform   = "true"
    }
}

# Using module outputs
resource "aws_instance" "app" {
    subnet_id = module.network.private_subnet_ids[0]
    
    tags = {
        Name = "App Server"
        VPC  = module.network.vpc_id
    }
}
```

## Advanced Module Patterns

### Conditional Resource Creation

You can make modules more flexible by allowing conditional resource creation:

```hcl
# variables.tf
variable "create_nat_gateway" {
    description = "Whether to create NAT Gateway"
    type        = bool
    default     = true
}

# main.tf
resource "aws_nat_gateway" "main" {
    count = var.create_nat_gateway ? 1 : 0
    
    allocation_id = aws_eip.nat[0].id
    subnet_id     = aws_subnet.public[0].id
    
    depends_on = [aws_internet_gateway.main]
    
    tags = merge(
        var.common_tags,
        {
            Name = "${var.environment}-nat"
        }
    )
}
```

### Dynamic Resource Generation

Modules can create resources dynamically based on input:

```hcl
# variables.tf
variable "subnet_configurations" {
    description = "Map of subnet configurations"
    type = map(object({
        cidr_block        = string
        availability_zone = string
        public           = bool
        tags             = map(string)
    }))
}

# main.tf
resource "aws_subnet" "this" {
    for_each = var.subnet_configurations
    
    vpc_id            = aws_vpc.main.id
    cidr_block        = each.value.cidr_block
    availability_zone = each.value.availability_zone
    
    tags = merge(
        var.common_tags,
        each.value.tags,
        {
            Name = "${var.environment}-${each.key}"
            Type = each.value.public ? "Public" : "Private"
        }
    )
}
```

### Module Composition

Modules can be composed of other modules:

```hcl
# modules/application/main.tf
module "network" {
    source = "../network"
    
    environment        = var.environment
    vpc_cidr          = var.vpc_cidr
    private_subnets   = var.private_subnets
    availability_zones = var.availability_zones
}

module "database" {
    source = "../database"
    
    subnet_ids = module.network.private_subnet_ids
    vpc_id     = module.network.vpc_id
}

module "application" {
    source = "../web-app"
    
    vpc_id             = module.network.vpc_id
    database_endpoint  = module.database.endpoint
    private_subnet_ids = module.network.private_subnet_ids
}
```

## Module Best Practices

### Documentation

Always include comprehensive documentation in your modules:

```hcl
# README.md for your module
# Network Module

This module creates a VPC with public and private subnets.

## Usage

```hcl
module "network" {
    source = "path/to/module"
    
    environment = "production"
    vpc_cidr    = "10.0.0.0/16"
}
```

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 1.0.0 |
| aws | >= 4.0.0 |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| environment | Environment name | `string` | n/a | yes |
| vpc_cidr | VPC CIDR block | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| vpc_id | The ID of the VPC |

```

### Error Handling

Implement proper validation and error handling:

```hcl
# variables.tf
variable "environment" {
    description = "Environment name"
    type        = string
    
    validation {
        condition     = contains(["dev", "staging", "prod"], var.environment)
        error_message = "Environment must be dev, staging, or prod."
    }
}

variable "vpc_cidr" {
    description = "VPC CIDR block"
    type        = string
    
    validation {
        condition     = can(cidrhost(var.vpc_cidr, 0))
        error_message = "Must be a valid IPv4 CIDR block."
    }
}
```

### State Management

Consider state management in your module design:

```hcl
# versions.tf
terraform {
    # Allow users to specify their own backend
    backend "s3" {}
}

# Example backend configuration in root module
terraform {
    backend "s3" {
        bucket = "terraform-state"
        key    = "network/terraform.tfstate"
        region = "us-west-2"
    }
}
```

