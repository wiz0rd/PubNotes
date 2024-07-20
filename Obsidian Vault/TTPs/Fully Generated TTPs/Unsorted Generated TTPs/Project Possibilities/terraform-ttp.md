# Infrastructure as Code with Terraform TTP

## Project Goal
The goal of this project is to implement Infrastructure as Code (IaC) using Terraform. By the end of this project, you will be able to define, version, and manage cloud infrastructure using code, enabling consistent and repeatable deployments across multiple environments.

## Overview
This TTP provides detailed instructions for setting up Terraform, creating infrastructure configurations, and managing cloud resources using code.

![Terraform Workflow](https://example.com/path/to/terraform_workflow.png)
*Figure 1: Terraform Workflow Diagram. Replace with an actual image showing the Terraform workflow: Write, Plan, Apply.*

## Prerequisites
- A cloud provider account (e.g., AWS, Azure, or GCP)
- Basic understanding of cloud services and infrastructure concepts
- Familiarity with command-line interfaces
- Git installed for version control

## Step-by-step Guide

### 1. Install Terraform

For Ubuntu/Debian:

```bash
# Add HashiCorp GPG key
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -

# Add HashiCorp repository
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"

# Update and install Terraform
sudo apt update && sudo apt install terraform
```

For other operating systems, follow the [official installation guide](https://learn.hashicorp.com/tutorials/terraform/install-cli).

Verify the installation:

```bash
terraform version
```

### 2. Configure Cloud Provider Credentials

For AWS (example):

```bash
# Install AWS CLI
sudo apt install awscli

# Configure AWS credentials
aws configure
```

Enter your AWS Access Key ID, Secret Access Key, default region, and output format when prompted.

### 3. Create a Terraform Configuration

Create a new directory for your Terraform project:

```bash
mkdir terraform-project
cd terraform-project
```

Create a file named `main.tf` with the following content (AWS example):

```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "terraform-example-instance"
  }
}
```

### 4. Initialize Terraform

In your project directory:

```bash
terraform init
```

This command initializes Terraform, downloads the necessary provider plugins, and sets up the backend.

### 5. Plan the Infrastructure

```bash
terraform plan
```

Review the plan to ensure it matches your expectations.

### 6. Apply the Configuration

```bash
terraform apply
```

Type 'yes' when prompted to create the resources.

### 7. Verify Resources

Log into your cloud provider's console to verify that the resources have been created as specified.

![AWS Console](https://example.com/path/to/aws_console.png)
*Figure 2: AWS Console showing the created EC2 instance. Replace with an actual screenshot of the AWS console showing the created resources.*

### 8. Use Terraform Modules

Create a new file named `modules.tf`:

```hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway = true
  enable_vpn_gateway = true

  tags = {
    Terraform = "true"
    Environment = "dev"
  }
}
```

Apply the changes:

```bash
terraform apply
```

### 9. Implement State Management

Set up remote state storage using an S3 bucket and DynamoDB for state locking:

Create a new file named `backend.tf`:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state-bucket"
    key            = "terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

Initialize the backend:

```bash
terraform init
```

### 10. Destroy Infrastructure

When you're done experimenting:

```bash
terraform destroy
```

Type 'yes' when prompted to destroy all resources.

## Advanced Topics

- Working with Terraform workspaces for managing multiple environments
- Implementing CI/CD pipelines for Terraform (e.g., using GitLab CI or GitHub Actions)
- Using Terraform Cloud for team collaboration and remote operations
- Implementing custom providers and modules

## Troubleshooting

- If `terraform apply` fails, check your cloud provider credentials and permissions
- For state inconsistencies, use `terraform refresh` to update the state
- If modules fail to download, check your internet connection and firewall settings

## Best Practices

1. Use version control (Git) for your Terraform configurations
2. Implement a naming convention for resources
3. Use variables and locals to make your code more reusable
4. Utilize data sources to fetch existing resource information
5. Implement proper state management and locking mechanisms
6. Use modules to encapsulate and reuse common infrastructure patterns

## Additional Resources

- [Terraform Documentation](https://www.terraform.io/docs/index.html)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [Terraform Module Registry](https://registry.terraform.io/)
- [Terraform Up & Running (Book)](https://www.oreilly.com/library/view/terraform-up/9781492046899/)
- [Awesome Terraform (GitHub)](https://github.com/shuaibiyy/awesome-terraform)

## Glossary

- **State**: Terraform's representation of your infrastructure's current condition
- **Plan**: A preview of changes Terraform will make to match your configuration
- **Apply**: The action of making your infrastructure match your configuration
- **Module**: A container for multiple resources that are used together
- **Provider**: A plugin that Terraform uses to interact with cloud providers, SaaS providers, and other APIs

