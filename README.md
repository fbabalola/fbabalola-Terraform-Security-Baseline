# Terraform Security Baseline

Infrastructure as Code for deploying secure AWS and Azure network configurations.

## Why I Built This

I kept seeing the same misconfigurations in cloud environments overly permissive security groups, SSH open to the world, no egress filtering. Instead of fixing these manually every time, I wrote Terraform configs that deploy secure-by-default network rules.

This isn't a production-ready enterprise solution. It's a starting point for locking down basic cloud infrastructure.

## What's In Here

```
terraform-security-baseline/
├── aws/
│   ├── main.tf                 # AWS security group with restricted access
│   ├── variables.tf            # Configurable parameters
│   └── outputs.tf              # Export security group ID
├── azure/
│   ├── main.tf                 # Azure NSG with deny-by-default rules
│   ├── variables.tf            
│   └── outputs.tf              
└── README.md
```

## Prerequisites

```bash
# Install Terraform (macOS)
brew install terraform

# Or download from terraform.io for other platforms
# https://developer.hashicorp.com/terraform/downloads

# Verify installation
terraform --version
# Should see: Terraform v1.x.x

# AWS CLI configured (for AWS configs)
aws configure
# Enter your Access Key, Secret Key, region

# Azure CLI configured (for Azure configs)
az login
```

## Quick Start - AWS

```bash
# Clone the repo
git clone https://github.com/fbabalola/terraform-security-baseline.git
cd terraform-security-baseline/aws

# Initialize Terraform (downloads AWS provider)
terraform init

# See what will be created BEFORE actually creating it
terraform plan

# If it looks good, apply
terraform apply

# Type 'yes' when prompted
# Note the security group ID in the output - you'll need it to attach to EC2 instances
```

## Quick Start - Azure

```bash
cd terraform-security-baseline/azure

terraform init
terraform plan
terraform apply
```

## Security Controls Implemented

### AWS Security Group

| Rule | Direction | Port | Source/Dest | Why |
|------|-----------|------|-------------|-----|
| SSH | Inbound | 22 | Your IP only | Not 0.0.0.0/0 - that's how boxes get popped |
| HTTPS | Inbound | 443 | 0.0.0.0/0 | Web traffic, encrypted only |
| HTTP | Inbound | 80 | 0.0.0.0/0 | Redirect to HTTPS in your app |
| All | Outbound | All | 0.0.0.0/0 | Default egress (tighten for prod) |

### Azure NSG

| Rule | Priority | Direction | Port | Source | Why |
|------|----------|-----------|------|--------|-----|
| DenyAllInbound | 4096 | Inbound | * | * | Default deny - explicit allow only |
| AllowSSH | 100 | Inbound | 22 | Your IP | Management access |
| AllowHTTPS | 110 | Inbound | 443 | * | Web traffic |
| AllowHTTP | 120 | Inbound | 80 | * | Redirect to 443 |

## Customizing

Edit `variables.tf` to change:

```hcl
# Your IP for SSH access (don't leave this as 0.0.0.0/0)
variable "allowed_ssh_cidr" {
  default = "YOUR.IP.HERE/32"
}

# AWS region
variable "aws_region" {
  default = "us-east-1"
}
```

## Common Issues

**"Error: No valid credential sources found"**
```bash
# AWS credentials not configured
aws configure
# Or export environment variables:
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"
```

**"Error: building account: getting authenticated object ID"**
```bash
# Azure not logged in
az login
az account set --subscription "Your Subscription Name"
```

**"Error: Invalid cidr_block"**
```bash
# You forgot to set your IP in variables.tf
# Find your IP:
curl ifconfig.me
# Then update allowed_ssh_cidr in variables.tf
```

## Destroying Resources

When you're done testing:

```bash
# This removes everything Terraform created
terraform destroy
# Type 'yes' to confirm

# Always destroy test resources to avoid charges
```

## What This Doesn't Cover

- VPC/VNet creation (assumes you have one)
- WAF rules
- DDoS protection
- Flow logs
- More granular egress filtering

These are more complex and depend on your architecture. This repo is just the network ACL baseline.

## Mapping to Compliance

| Control | Framework | How This Helps |
|---------|-----------|----------------|
| Restrict network access | NIST 800-53 SC-7 | Security groups limit ingress |
| Least privilege | NIST 800-53 AC-6 | Only required ports open |
| Default deny | CIS Benchmark | Azure NSG denies unlisted traffic |

## Author

Firebami Babalola  
Security Operations Analyst | SC-200 | Security+

Built this while learning IaC for security automation. Feedback welcome.
