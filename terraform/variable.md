# Understanding Terraform Variables

## Introduction to Terraform Variables

Variables in Terraform serve as the foundation for creating flexible and reusable infrastructure code. They allow you to parameterize your configurations, making them adaptable to different environments and use cases. Think of variables as the knobs and dials that control how your infrastructure gets built, without changing the underlying configuration code.

## Variable Declaration and Types

### Basic Variable Declaration

At its most basic level, a variable declaration tells Terraform that a particular variable exists and may optionally provide default values and constraints. Here's the anatomy of a variable declaration:

```hcl
variable "instance_type" {
    description = "The type of EC2 instance to launch"
    type        = string
    default     = "t2.micro"
    
    validation {
        condition     = can(regex("^t[23].", var.instance_type))
        error_message = "Instance type must be a t2 or t3 instance."
    }
}
```

Let's break down each component:

1. The `description` field provides documentation about the variable's purpose
2. The `type` field specifies what kind of value the variable accepts
3. The `default` field provides a fallback value if none is specified
4. The `validation` block ensures the variable meets certain criteria

### Understanding Variable Types

Terraform supports several variable types, each serving different purposes:

#### Primitive Types

```hcl
# String variable
variable "environment" {
    type    = string
    default = "development"
}

# Number variable
variable "instance_count" {
    type    = number
    default = 1
}

# Boolean variable
variable "enable_monitoring" {
    type    = bool
    default = true
}
```

#### Complex Types

**List Type:**

```hcl
# Simple list
variable "availability_zones" {
    type    = list(string)
    default = ["us-west-2a", "us-west-2b", "us-west-2c"]
}

# List of objects
variable "subnet_configurations" {
    type = list(object({
        cidr_block        = string
        availability_zone = string
        public           = bool
    }))
    default = [
        {
            cidr_block        = "10.0.1.0/24"
            availability_zone = "us-west-2a"
            public           = true
        },
        {
            cidr_block        = "10.0.2.0/24"
            availability_zone = "us-west-2b"
            public           = false
        }
    ]
}
```

**Map Type:**

```hcl
# Simple map
variable "instance_types" {
    type = map(string)
    default = {
        development = "t2.micro"
        staging     = "t2.medium"
        production  = "t2.large"
    }
}

# Map of objects
variable "resource_tags" {
    type = map(object({
        owner       = string
        department  = string
        cost_center = string
    }))
    default = {
        web = {
            owner       = "web-team"
            department  = "engineering"
            cost_center = "web-123"
        },
        db = {
            owner       = "db-team"
            department  = "engineering"
            cost_center = "db-123"
        }
    }
}
```

**Set Type:**

```hcl
variable "allowed_ports" {
    type    = set(number)
    default = [22, 80, 443]
}
```

## Variable Usage Patterns

### Basic Variable Usage

Variables can be referenced throughout your Terraform configuration using the `var.` prefix:

```hcl
resource "aws_instance" "web" {
    instance_type = var.instance_type
    ami           = var.ami_id
    
    tags = {
        Environment = var.environment
        Name        = "${var.project_name}-web-server"
    }
}
```

### Variable Interpolation

Variables can be combined with strings and other variables using interpolation:

```hcl
locals {
    # Simple interpolation
    instance_name = "${var.environment}-${var.component}-server"
    
    # Complex interpolation
    tags = {
        Name        = local.instance_name
        Environment = var.environment
        Component   = var.component
        FullName    = "Project ${var.project_name} ${local.instance_name}"
    }
}
```

### Variable Validation

Complex validation rules ensure variables meet specific criteria:

```hcl
variable "environment" {
    type        = string
    description = "The deployment environment"
    
    validation {
        # Basic validation
        condition     = contains(["dev", "staging", "prod"], var.environment)
        error_message = "Environment must be dev, staging, or prod."
    }
}

variable "vpc_cidr" {
    type        = string
    description = "CIDR block for the VPC"
    
    validation {
        # Complex validation with regex
        condition     = can(regex("^([0-9]{1,3}\\.){3}[0-9]{1,3}/[0-9]{1,2}$", var.vpc_cidr))
        error_message = "VPC CIDR must be a valid IPv4 CIDR block."
    }
    
    validation {
        # Additional validation for CIDR range
        condition     = cidrhost(var.vpc_cidr, 0) != cidrhost(var.vpc_cidr, -1)
        error_message = "VPC CIDR block must be large enough to contain subnets."
    }
}
```

## Variable Input Methods

### Command Line Variables

Variables can be set when running Terraform commands:

```bash
terraform apply -var="environment=production" -var="instance_type=t2.medium"
```

### Variable Files

Variables can be defined in separate files (typically named `terraform.tfvars` or `*.auto.tfvars`):

```hcl
# terraform.tfvars
environment    = "production"
instance_type  = "t2.medium"
instance_count = 3
vpc_cidr       = "10.0.0.0/16"

# Production specific tags
resource_tags = {
    owner        = "operations"
    environment  = "production"
    cost_center  = "ops-123"
}
```

### Environment Variables

Variables can be set using environment variables prefixed with `TF_VAR_`:

```bash
export TF_VAR_environment="production"
export TF_VAR_instance_type="t2.medium"
```

## Variable Best Practices

### Organization and Naming

Follow consistent naming conventions:

```hcl
# Group related variables
variable "vpc_cidr" {}
variable "vpc_name" {}
variable "vpc_enable_dns" {}

# Use descriptive names
variable "primary_database_instance_type" {}
variable "read_replica_instance_count" {}
```

### Documentation

Provide comprehensive documentation for variables:

```hcl
variable "backup_retention_days" {
    description = <<EOF
The number of days to retain automated backups. 
Setting this value to 0 disables automated backups.
Minimum value is 0, maximum value is 35.
EOF
    type        = number
    default     = 7
    
    validation {
        condition     = var.backup_retention_days >= 0 && var.backup_retention_days <= 35
        error_message = "Backup retention must be between 0 and 35 days."
    }
}
```

### Security Considerations

Handle sensitive variables appropriately:

```hcl
variable "database_password" {
    type        = string
    sensitive   = true
    description = "Password for the database instance"
    
    validation {
        condition     = length(var.database_password) >= 16
        error_message = "Database password must be at least 16 characters long."
    }
}
```

## Advanced Variable Techniques

### Dynamic Variable Usage

Using variables in dynamic blocks:

```hcl
resource "aws_security_group" "example" {
    name = "example-security-group"
    
    dynamic "ingress" {
        for_each = var.service_ports
        content {
            from_port   = ingress.value
            to_port     = ingress.value
            protocol    = "tcp"
            cidr_blocks = var.allowed_cidr_blocks
        }
    }
}
```

### Variable Type Conversion

Converting between variable types:

```hcl
locals {
    # Convert string to number
    port_number = tonumber(var.port_string)
    
    # Convert list to set
    unique_tags = toset(var.tag_list)
    
    # Convert string to list
    subnet_list = split(",", var.subnet_string)
    
    # Convert map to list of objects
    instance_configs = [
        for name, config in var.instance_map : {
            name = name
            type = config.instance_type
            zone = config.availability_zone
        }
    ]
}
```

### Complex Variable Transformations

Working with complex variable structures:

```hcl
locals {
    # Transform list of objects
    formatted_instances = [
        for instance in var.instance_configurations : {
            name = lower(instance.name)
            type = contains(["prod", "staging"], var.environment) ? instance.production_type : instance.development_type
            tags = merge(var.common_tags, instance.specific_tags)
        }
    ]
    
    # Create map from list
    instance_map = {
        for instance in local.formatted_instances : instance.name => instance
    }
}
```
