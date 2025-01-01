# Understanding Terraform Data Sources

## Introduction to Data Sources

Data sources in Terraform allow you to query and fetch information from your existing infrastructure or external services. Think of data sources as "read-only" views into resources that exist outside of your current Terraform configuration. They're like windows that let you peek into your infrastructure and use that information in your Terraform code.

## Basic Data Source Structure

A data source follows this fundamental structure:

```hcl
data "provider_resource" "name" {
    # Query parameters go here
    filter_parameter = "value"
}

# Reference the data using: data.provider_resource.name.attribute
```

## Common Data Source Use Cases

### Fetching Existing VPC Information

Let's say you need to deploy resources into an existing VPC. Instead of hardcoding the VPC ID, you can query it:

```hcl
data "aws_vpc" "existing" {
    # Find VPC by tag
    filter {
        name   = "tag:Environment"
        values = ["Production"]
    }
}

# Use the VPC information
resource "aws_subnet" "new_subnet" {
    vpc_id     = data.aws_vpc.existing.id
    cidr_block = "10.0.1.0/24"
    
    tags = {
        Name = "New Subnet in ${data.aws_vpc.existing.tags["Name"]}"
    }
}
```

### Finding the Latest AMI

When launching EC2 instances, you often need the latest AMI. Here's how to find it dynamically:

```hcl
data "aws_ami" "ubuntu_latest" {
    most_recent = true
    owners      = ["099720109477"] # Canonical's AWS account ID
    
    filter {
        name   = "name"
        values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
    }
    
    filter {
        name   = "virtualization-type"
        values = ["hvm"]
    }
}

resource "aws_instance" "web_server" {
    ami           = data.aws_ami.ubuntu_latest.id
    instance_type = "t3.micro"
    
    tags = {
        Name        = "Web Server"
        AMI_Name    = data.aws_ami.ubuntu_latest.name
        AMI_Version = data.aws_ami.ubuntu_latest.description
    }
}
```

### Retrieving Availability Zones

When you need to spread resources across availability zones:

```hcl
data "aws_availability_zones" "available" {
    state = "available"
    
    # Exclude Local Zones
    filter {
        name   = "opt-in-status"
        values = ["opt-in-not-required"]
    }
}

# Create subnets across all AZs
resource "aws_subnet" "multi_az" {
    count             = length(data.aws_availability_zones.available.names)
    vpc_id            = aws_vpc.main.id
    availability_zone = data.aws_availability_zones.available.names[count.index]
    cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 4, count.index)
    
    tags = {
        Name = "Subnet-${data.aws_availability_zones.available.names[count.index]}"
    }
}
```

### Accessing Remote State Data

You can use data sources to access state from other Terraform configurations:

```hcl
# Access state from another Terraform project
data "terraform_remote_state" "network" {
    backend = "s3"
    
    config = {
        bucket = "terraform-state-bucket"
        key    = "network/terraform.tfstate"
        region = "us-west-2"
    }
}

# Use the network information
resource "aws_instance" "application" {
    subnet_id = data.terraform_remote_state.network.outputs.private_subnet_ids[0]
    vpc_id    = data.terraform_remote_state.network.outputs.vpc_id
    
    tags = {
        Name = "Application Server"
        VPC  = data.terraform_remote_state.network.outputs.vpc_name
    }
}
```

## Advanced Data Source Techniques

### Combining Multiple Data Sources

You can combine multiple data sources to build complex configurations:

```hcl
# Find the security group
data "aws_security_group" "selected" {
    tags = {
        Environment = var.environment
        Role        = "web"
    }
}

# Find the subnet
data "aws_subnet" "selected" {
    filter {
        name   = "tag:Tier"
        values = ["Application"]
    }
    
    filter {
        name   = "vpc-id"
        values = [data.aws_vpc.existing.id]
    }
}

# Use both data sources
resource "aws_instance" "application" {
    ami                    = data.aws_ami.ubuntu_latest.id
    instance_type          = "t3.micro"
    subnet_id              = data.aws_subnet.selected.id
    vpc_security_group_ids = [data.aws_security_group.selected.id]
    
    tags = {
        Name = "Application Server"
    }
}
```

### Using Data Sources with Dynamic Blocks

Data sources can power dynamic configurations:

```hcl
data "aws_ip_ranges" "aws_ec2_ip_ranges" {
    regions  = ["us-east-1", "us-west-2"]
    services = ["ec2"]
}

resource "aws_security_group" "aws_service_access" {
    name        = "aws-service-access"
    description = "Allow access from AWS EC2 IP ranges"
    
    dynamic "ingress" {
        for_each = data.aws_ip_ranges.aws_ec2_ip_ranges.cidr_blocks
        content {
            from_port   = 443
            to_port     = 443
            protocol    = "tcp"
            cidr_blocks = [ingress.value]
            
            description = "HTTPS from AWS EC2 IP range"
        }
    }
}
```

## Best Practices for Data Sources

### Error Handling and Validation

Always validate data source results:

```hcl
locals {
    # Ensure we found exactly one VPC
    vpc_count = length(data.aws_vpc.existing[*].id)
    
    # Validate the result
    validate_vpc = (
        local.vpc_count == 1 
        ? null 
        : file("ERROR: Found ${local.vpc_count} VPCs, expected exactly 1")
    )
}
```

### Using Data Source References

Structure your code to make data source usage clear:

```hcl
locals {
    # Collect all data source references in one place
    infrastructure = {
        vpc_id         = data.aws_vpc.existing.id
        subnet_ids     = data.aws_subnet_ids.available.ids
        ami_id         = data.aws_ami.ubuntu_latest.id
        instance_profile = data.aws_iam_instance_profile.web_profile.name
    }
}

# Use the local values
resource "aws_instance" "web" {
    ami                  = local.infrastructure.ami_id
    instance_type        = "t3.micro"
    subnet_id            = local.infrastructure.subnet_ids[0]
    iam_instance_profile = local.infrastructure.instance_profile
}
```

### Documentation and Maintainability

Always document your data source usage:

```hcl
data "aws_vpc" "existing" {
    # Find the VPC used for our application tier
    # This VPC should have the following tags:
    # - Environment = Production
    # - Tier = Application
    filter {
        name   = "tag:Environment"
        values = ["Production"]
    }
    
    filter {
        name   = "tag:Tier"
        values = ["Application"]
    }
}
```

## Common Pitfalls and Solutions

### Dealing with Multiple Results

When a data source might return multiple results:

```hcl
# Find all subnets
data "aws_subnet_ids" "all" {
    vpc_id = data.aws_vpc.existing.id
}

# Validate the count
locals {
    subnet_count = length(data.aws_subnet_ids.all.ids)
    validate_subnets = (
        local.subnet_count >= 2 
        ? null 
        : file("ERROR: Need at least 2 subnets, found ${local.subnet_count}")
    )
}
```

### Handling Dependencies

Use depends_on when necessary:

```hcl
data "aws_instance" "web_server" {
    filter {
        name   = "tag:Role"
        values = ["WebServer"]
    }
    
    depends_on = [
        aws_instance.web  # Wait for instance to be created
    ]
}
```

### Refresh Behavior

Understanding data source refresh behavior:

```hcl
# Force data source refresh
data "aws_security_group" "selected" {
    # This will be refreshed on every apply
    id = var.security_group_id
    
    lifecycle {
        postcondition {
            condition     = self.vpc_id == data.aws_vpc.existing.id
            error_message = "Security group must be in the selected VPC"
        }
    }
}
```
