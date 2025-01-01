# Understanding Terraform .tfvars Files

## What Are .tfvars Files?

Terraform .tfvars files are special files that contain variable values. They allow you to separate your variable definitions (what variables exist and their types) from their actual values. This separation is crucial for maintaining clean, reusable, and secure infrastructure code.

## Basic Structure and Usage

Let's start with a simple example to understand how .tfvars files work:

```hcl
# First, in your main.tf or variables.tf, you define your variables:
variable "environment" {
    type        = string
    description = "The deployment environment"
}

variable "instance_type" {
    type        = string
    description = "The type of EC2 instance to launch"
}

variable "instance_count" {
    type        = number
    description = "Number of instances to create"
}

# Then, in your terraform.tfvars file:
environment    = "production"
instance_type  = "t2.micro"
instance_count = 3
```

## Different Types of Variable Files

Terraform recognizes several types of variable files, each with its own purpose:

1. **terraform.tfvars or *.auto.tfvars**
   - Automatically loaded by Terraform
   - Great for default values

   ```hcl
   # terraform.tfvars
   region        = "us-west-2"
   instance_type = "t2.micro"
   ```

2. **environment-specific.tfvars**
   - Manually specified when running Terraform
   - Perfect for environment-specific values

   ```hcl
   # production.tfvars
   environment    = "production"
   instance_type  = "t2.medium"
   instance_count = 3
   ```

   ```hcl
   # development.tfvars
   environment    = "development"
   instance_type  = "t2.micro"
   instance_count = 1
   ```

## Practical Examples

### Example 1: Basic Infrastructure Configuration

```hcl
# variables.tf
variable "vpc_settings" {
    type = object({
        cidr_block = string
        name       = string
    })
    description = "VPC configuration settings"
}

variable "environment" {
    type = string
}

# terraform.tfvars
vpc_settings = {
    cidr_block = "10.0.0.0/16"
    name       = "main-vpc"
}
environment = "staging"
```

### Example 2: Environment-Specific Configurations

```hcl
# variables.tf
variable "database_config" {
    type = object({
        instance_class    = string
        allocated_storage = number
        multi_az         = bool
    })
}

# production.tfvars
database_config = {
    instance_class    = "db.r5.large"
    allocated_storage = 100
    multi_az         = true
}

# staging.tfvars
database_config = {
    instance_class    = "db.t3.medium"
    allocated_storage = 50
    multi_az         = false
}
```

### Example 3: Complex Infrastructure Settings

```hcl
# variables.tf
variable "application_config" {
    type = object({
        name = string
        instances = map(object({
            type  = string
            count = number
            zone  = string
        }))
        security = object({
            enable_encryption = bool
            allowed_ips      = list(string)
        })
    })
}

# app-config.tfvars
application_config = {
    name = "web-platform"
    instances = {
        web = {
            type  = "t3.medium"
            count = 3
            zone  = "us-west-2a"
        }
        api = {
            type  = "t3.large"
            count = 2
            zone  = "us-west-2b"
        }
    }
    security = {
        enable_encryption = true
        allowed_ips      = ["10.0.0.0/8", "172.16.0.0/12"]
    }
}
```

## Best Practices for Using .tfvars Files

1. **Environment Organization**

   ```plaintext
   project/
   ├── main.tf
   ├── variables.tf
   ├── terraform.tfvars      # Default values
   ├── production.tfvars     # Production environment
   ├── staging.tfvars        # Staging environment
   └── development.tfvars    # Development environment
   ```

2. **Sensitive Information Handling**

   ```hcl
   # secrets.tfvars (This should NEVER be committed to version control)
   database_password = "very-secret-password"
   api_keys = {
    primary   = "key1234"
    secondary = "key5678"
   }
   ```

3. **Using Multiple Variable Files**
   You can use multiple .tfvars files together:

   ```bash
   terraform apply \
     -var-file="common.tfvars" \
     -var-file="production.tfvars" \
     -var-file="secrets.tfvars"
   ```

## Advanced Usage Patterns

### Dynamic Variable Generation

You can generate .tfvars files dynamically using scripts:

```bash
#!/bin/bash
# generate-tfvars.sh
cat << EOF > environment.tfvars
environment     = "${ENV_NAME}"
instance_count  = ${INSTANCE_COUNT}
instance_type   = "${INSTANCE_TYPE}"
region          = "${AWS_REGION}"
EOF
```

### Variable File Templating

You can use template files to generate .tfvars files:

```hcl
# template.tfvars.tpl
environment = "${env}"
region      = "${region}"
tags = {
    Environment = "${env}"
    Region      = "${region}"
    Team        = "${team}"
}
```

### Complex Data Structures

```hcl
# complex-config.tfvars
networking = {
    vpc = {
        cidr = "10.0.0.0/16"
        subnets = {
            public = {
                "us-west-2a" = "10.0.1.0/24"
                "us-west-2b" = "10.0.2.0/24"
            }
            private = {
                "us-west-2a" = "10.0.10.0/24"
                "us-west-2b" = "10.0.11.0/24"
            }
        }
    }
    security_groups = {
        web = {
            name = "web-sg"
            rules = [
                {
                    port        = 80
                    protocol    = "tcp"
                    cidr_blocks = ["0.0.0.0/0"]
                },
                {
                    port        = 443
                    protocol    = "tcp"
                    cidr_blocks = ["0.0.0.0/0"]
                }
            ]
        }
    }
}
```
