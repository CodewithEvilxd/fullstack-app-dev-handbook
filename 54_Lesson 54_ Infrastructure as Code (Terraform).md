# Lesson 54: Infrastructure as Code (Terraform)

## 🎯 **Learning Objectives**
- Master Infrastructure as Code (IaC) principles using Terraform
- Create and manage cloud infrastructure for React Native applications
- Implement scalable and maintainable infrastructure patterns
- Automate infrastructure provisioning and deployment
- Handle multi-environment infrastructure management

## 📚 **Table of Contents**
1. [IaC Fundamentals](#iac-fundamentals)
2. [Terraform Basics](#terraform-basics)
3. [AWS Infrastructure Setup](#aws-infrastructure-setup)
4. [Database Infrastructure](#database-infrastructure)
5. [CI/CD Infrastructure](#cicd-infrastructure)
6. [Monitoring Infrastructure](#monitoring-infrastructure)
7. [Security Infrastructure](#security-infrastructure)
8. [Multi-Environment Management](#multi-environment-management)
9. [Cost Optimization](#cost-optimization)
10. [Practical Examples](#practical-examples)

---

## 🏗️ **IaC Fundamentals**

### **What is Infrastructure as Code?**
Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools.

### **Benefits of IaC**
- **Consistency**: Ensure identical environments across development, staging, and production
- **Version Control**: Track infrastructure changes in Git
- **Scalability**: Easily scale infrastructure up or down
- **Reliability**: Reduce human error in infrastructure setup
- **Speed**: Quickly provision and tear down environments
- **Documentation**: Infrastructure configuration serves as documentation

### **IaC Tools Comparison**
| Tool | Language | Cloud Support | State Management | Learning Curve |
|------|----------|---------------|------------------|----------------|
| Terraform | HCL | Multi-cloud | Local/Remote | Moderate |
| CloudFormation | JSON/YAML | AWS only | AWS managed | Steep |
| ARM Templates | JSON | Azure only | Azure managed | Steep |
| Pulumi | Programming languages | Multi-cloud | Local/Remote | Gentle |

---

## 🚀 **Terraform Basics**

### **Installation and Setup**
```bash
# Install Terraform (macOS with Homebrew)
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Install Terraform (Linux)
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com jammy main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Verify installation
terraform version

# Initialize Terraform project
mkdir terraform-project
cd terraform-project
terraform init
```

### **Basic Terraform Structure**
```
terraform-project/
├── main.tf              # Main configuration
├── variables.tf         # Input variables
├── outputs.tf          # Output values
├── terraform.tfvars    # Variable values
├── providers.tf        # Provider configuration
├── backend.tf          # State backend configuration
└── modules/            # Reusable modules
    ├── vpc/
    ├── ec2/
    └── rds/
```

### **Basic Configuration**
```hcl
# main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }

  # State backend configuration
  backend "s3" {
    bucket = "my-terraform-state-bucket"
    key    = "terraform/state"
    region = "us-east-1"
  }
}

# Provider configuration
provider "aws" {
  region = var.aws_region

  # Optional: Assume role for cross-account access
  assume_role {
    role_arn = var.assume_role_arn
  }
}

# VPC resource
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name        = "${var.project_name}-vpc"
    Environment = var.environment
    Project     = var.project_name
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name        = "${var.project_name}-igw"
    Environment = var.environment
  }
}
```

```hcl
# variables.tf
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "react-native-app"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be one of: dev, staging, prod"
  }
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "assume_role_arn" {
  description = "ARN of the role to assume"
  type        = string
  default     = null
}
```

```hcl
# outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "vpc_cidr" {
  description = "VPC CIDR block"
  value       = aws_vpc.main.cidr_block
}

output "internet_gateway_id" {
  description = "Internet Gateway ID"
  value       = aws_internet_gateway.main.id
}
```

```hcl
# terraform.tfvars
aws_region    = "us-east-1"
project_name  = "my-react-native-app"
environment   = "dev"
vpc_cidr      = "10.0.0.0/16"
```

### **Terraform Commands**
```bash
# Initialize Terraform
terraform init

# Format configuration files
terraform fmt

# Validate configuration
terraform validate

# Plan changes
terraform plan

# Apply changes
terraform apply

# Destroy infrastructure
terraform destroy

# Show current state
terraform show

# List resources
terraform state list

# Import existing resources
terraform import aws_instance.example i-1234567890abcdef0

# Refresh state
terraform refresh

# Output values
terraform output

# Create workspace
terraform workspace create dev
terraform workspace select dev
terraform workspace list
```

---

## ☁️ **AWS Infrastructure Setup**

### **VPC and Networking**
```hcl
# modules/vpc/main.tf
resource "aws_vpc" "main" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(var.tags, {
    Name = "${var.name}-vpc"
  })
}

resource "aws_subnet" "public" {
  count             = length(var.public_subnets)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnets[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = merge(var.tags, {
    Name = "${var.name}-public-${count.index + 1}"
    Type = "Public"
  })
}

resource "aws_subnet" "private" {
  count             = length(var.private_subnets)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnets[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = merge(var.tags, {
    Name = "${var.name}-private-${count.index + 1}"
    Type = "Private"
  })
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = merge(var.tags, {
    Name = "${var.name}-igw"
  })
}

resource "aws_nat_gateway" "main" {
  count         = var.enable_nat_gateway ? 1 : 0
  allocation_id = aws_eip.nat[0].id
  subnet_id     = aws_subnet.public[0].id

  tags = merge(var.tags, {
    Name = "${var.name}-nat"
  })

  depends_on = [aws_internet_gateway.main]
}

resource "aws_eip" "nat" {
  count = var.enable_nat_gateway ? 1 : 0
  vpc   = true

  tags = merge(var.tags, {
    Name = "${var.name}-nat-eip"
  })
}

# Route Tables
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = merge(var.tags, {
    Name = "${var.name}-public-rt"
  })
}

resource "aws_route_table" "private" {
  count  = var.enable_nat_gateway ? 1 : 0
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[0].id
  }

  tags = merge(var.tags, {
    Name = "${var.name}-private-rt"
  })
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count          = length(var.public_subnets)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count          = length(var.private_subnets)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[0].id
}
```

```hcl
# modules/vpc/variables.tf
variable "name" {
  description = "Name of the VPC"
  type        = string
}

variable "cidr_block" {
  description = "CIDR block for VPC"
  type        = string
}

variable "public_subnets" {
  description = "List of public subnet CIDR blocks"
  type        = list(string)
}

variable "private_subnets" {
  description = "List of private subnet CIDR blocks"
  type        = list(string)
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

variable "enable_nat_gateway" {
  description = "Enable NAT Gateway for private subnets"
  type        = bool
  default     = true
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

### **EC2 Instances for React Native Backend**
```hcl
# modules/ec2/main.tf
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

resource "aws_instance" "web" {
  count         = var.instance_count
  ami           = data.aws_ami.amazon_linux.id
  instance_type = var.instance_type

  subnet_id                   = var.subnet_id
  vpc_security_group_ids      = [aws_security_group.web.id]
  associate_public_ip_address = var.associate_public_ip
  key_name                    = var.key_name

  user_data = templatefile("${path.module}/user-data.sh", {
    environment = var.environment
    db_host     = var.db_host
    db_name     = var.db_name
    db_user     = var.db_user
    db_password = var.db_password
  })

  tags = merge(var.tags, {
    Name = "${var.name}-web-${count.index + 1}"
  })

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_security_group" "web" {
  name_prefix = "${var.name}-web-"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = var.allowed_ssh_cidr_blocks
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(var.tags, {
    Name = "${var.name}-web-sg"
  })
}

resource "aws_eip" "web" {
  count    = var.associate_public_ip ? var.instance_count : 0
  instance = aws_instance.web[count.index].id
  vpc      = true

  tags = merge(var.tags, {
    Name = "${var.name}-web-eip-${count.index + 1}"
  })
}
```

```bash
# modules/ec2/user-data.sh
#!/bin/bash
# Install Node.js and React Native backend

# Update system
yum update -y

# Install Node.js
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
nvm install 18
nvm use 18

# Install PM2 for process management
npm install -g pm2

# Create application directory
mkdir -p /var/app
cd /var/app

# Clone repository (replace with your repo)
# git clone https://github.com/your-org/your-app.git .

# Install dependencies
npm install

# Create environment file
cat > .env << EOF
NODE_ENV=${environment}
PORT=3000
DATABASE_URL=postgresql://${db_user}:${db_password}@${db_host}:5432/${db_name}
JWT_SECRET=your-jwt-secret
EOF

# Build application
npm run build

# Start application with PM2
pm2 start ecosystem.config.js --env ${environment}
pm2 startup
pm2 save
```

---

## 🗄️ **Database Infrastructure**

### **RDS PostgreSQL Setup**
```hcl
# modules/rds/main.tf
resource "aws_db_instance" "main" {
  identifier = var.identifier

  engine         = "postgres"
  engine_version = var.engine_version
  instance_class = var.instance_class

  allocated_storage     = var.allocated_storage
  max_allocated_storage = var.max_allocated_storage
  storage_type          = "gp2"

  db_name  = var.db_name
  username = var.username
  password = var.password

  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name

  backup_retention_period = var.backup_retention_period
  backup_window           = var.backup_window
  maintenance_window      = var.maintenance_window

  multi_az               = var.multi_az
  publicly_accessible    = false
  skip_final_snapshot    = var.skip_final_snapshot
  final_snapshot_identifier = var.final_snapshot_identifier

  tags = merge(var.tags, {
    Name = "${var.identifier}-rds"
  })
}

resource "aws_db_subnet_group" "main" {
  name       = "${var.identifier}-subnet-group"
  subnet_ids = var.subnet_ids

  tags = merge(var.tags, {
    Name = "${var.identifier}-subnet-group"
  })
}

resource "aws_security_group" "rds" {
  name_prefix = "${var.identifier}-rds-"
  vpc_id      = var.vpc_id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = var.allowed_security_groups
  }

  tags = merge(var.tags, {
    Name = "${var.identifier}-rds-sg"
  })
}

# Read replica (optional)
resource "aws_db_instance" "replica" {
  count = var.create_replica ? 1 : 0

  identifier = "${var.identifier}-replica"

  instance_class = var.replica_instance_class
  replicate_source_db = aws_db_instance.main.identifier

  vpc_security_group_ids = [aws_security_group.rds.id]

  backup_retention_period = 0
  skip_final_snapshot     = true

  tags = merge(var.tags, {
    Name = "${var.identifier}-replica"
  })
}
```

### **DynamoDB for NoSQL Data**
```hcl
# modules/dynamodb/main.tf
resource "aws_dynamodb_table" "main" {
  name           = var.table_name
  billing_mode   = var.billing_mode
  hash_key       = var.hash_key
  range_key      = var.range_key

  # Attributes
  dynamic "attribute" {
    for_each = var.attributes
    content {
      name = attribute.value.name
      type = attribute.value.type
    }
  }

  # Global Secondary Indexes
  dynamic "global_secondary_index" {
    for_each = var.global_secondary_indexes
    content {
      name               = global_secondary_index.value.name
      hash_key           = global_secondary_index.value.hash_key
      range_key          = global_secondary_index.value.range_key
      projection_type    = global_secondary_index.value.projection_type
      read_capacity      = var.billing_mode == "PROVISIONED" ? global_secondary_index.value.read_capacity : null
      write_capacity     = var.billing_mode == "PROVISIONED" ? global_secondary_index.value.write_capacity : null
    }
  }

  # Local Secondary Indexes
  dynamic "local_secondary_index" {
    for_each = var.local_secondary_indexes
    content {
      name               = local_secondary_index.value.name
      range_key          = local_secondary_index.value.range_key
      projection_type    = local_secondary_index.value.projection_type
    }
  }

  # Auto scaling (for PROVISIONED billing mode)
  dynamic "auto_scaling" {
    for_each = var.billing_mode == "PROVISIONED" ? [1] : []
    content {
      enabled = true
      target = 70

      min_capacity = var.min_read_capacity
      max_capacity = var.max_read_capacity
    }
  }

  # Point-in-time recovery
  point_in_time_recovery {
    enabled = var.enable_point_in_time_recovery
  }

  # Server-side encryption
  server_side_encryption {
    enabled = true
  }

  tags = merge(var.tags, {
    Name = var.table_name
  })
}

# DynamoDB Auto Scaling Policy
resource "aws_appautoscaling_target" "read" {
  count = var.billing_mode == "PROVISIONED" ? 1 : 0

  max_capacity       = var.max_read_capacity
  min_capacity       = var.min_read_capacity
  resource_id        = "table/${aws_dynamodb_table.main.name}"
  scalable_dimension = "dynamodb:table:ReadCapacityUnits"
  service_namespace  = "dynamodb"
}

resource "aws_appautoscaling_policy" "read" {
  count = var.billing_mode == "PROVISIONED" ? 1 : 0

  name               = "${var.table_name}-read-policy"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.read[0].resource_id
  scalable_dimension = aws_appautoscaling_target.read[0].scalable_dimension
  service_namespace  = aws_appautoscaling_target.read[0].service_namespace

  target_tracking_scaling_policy_configuration {
    target_value = 70.0
    predefined_metric_specification {
      predefined_metric_type = "DynamoDBReadCapacityUtilization"
    }
  }
}
```

---

## 🔄 **CI/CD Infrastructure**

### **CodeBuild for Build Pipelines**
```hcl
# modules/codebuild/main.tf
resource "aws_codebuild_project" "main" {
  name          = var.project_name
  description   = var.description
  build_timeout = var.build_timeout
  service_role  = aws_iam_role.codebuild.arn

  artifacts {
    type = "NO_ARTIFACTS"
  }

  cache {
    type     = "S3"
    location = aws_s3_bucket.cache.bucket
  }

  environment {
    compute_type                = var.compute_type
    image                       = var.build_image
    type                        = "LINUX_CONTAINER"
    image_pull_credentials_type = "CODEBUILD"
    privileged_mode             = var.privileged_mode

    dynamic "environment_variable" {
      for_each = var.environment_variables
      content {
        name  = environment_variable.value.name
        value = environment_variable.value.value
        type  = environment_variable.value.type
      }
    }
  }

  logs_config {
    cloudwatch_logs {
      group_name  = aws_cloudwatch_log_group.codebuild.name
      stream_name = aws_cloudwatch_log_stream.codebuild.name
    }
  }

  source {
    type            = "GITHUB"
    location        = var.github_repository_url
    git_clone_depth = 1
    buildspec       = var.buildspec_file
  }

  vpc_config {
    vpc_id             = var.vpc_id
    subnets            = var.subnet_ids
    security_group_ids = [aws_security_group.codebuild.id]
  }

  tags = merge(var.tags, {
    Name = var.project_name
  })
}

resource "aws_codebuild_webhook" "main" {
  count        = var.enable_webhook ? 1 : 0
  project_name = aws_codebuild_project.main.name

  filter_group {
    filter {
      type    = "EVENT"
      pattern = "PUSH"
    }

    filter {
      type    = "HEAD_REF"
      pattern = var.webhook_branch_filter
    }
  }
}

# IAM Role for CodeBuild
resource "aws_iam_role" "codebuild" {
  name = "${var.project_name}-codebuild-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "codebuild.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "codebuild" {
  role       = aws_iam_role.codebuild.name
  policy_arn = "arn:aws:iam::aws:policy/AdministratorAccess" # Use least privilege in production
}

# S3 bucket for build cache
resource "aws_s3_bucket" "cache" {
  bucket = "${var.project_name}-codebuild-cache"

  tags = merge(var.tags, {
    Name = "${var.project_name}-cache"
  })
}

# CloudWatch Logs
resource "aws_cloudwatch_log_group" "codebuild" {
  name              = "/aws/codebuild/${var.project_name}"
  retention_in_days = 30

  tags = merge(var.tags, {
    Name = "${var.project_name}-logs"
  })
}

resource "aws_cloudwatch_log_stream" "codebuild" {
  name           = "build"
  log_group_name = aws_cloudwatch_log_group.codebuild.name
}

# Security Group
resource "aws_security_group" "codebuild" {
  name_prefix = "${var.project_name}-codebuild-"
  vpc_id      = var.vpc_id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(var.tags, {
    Name = "${var.project_name}-codebuild-sg"
  })
}
```

### **CodePipeline for Complete CI/CD**
```hcl
# modules/codepipeline/main.tf
resource "aws_codepipeline" "main" {
  name     = var.pipeline_name
  role_arn = aws_iam_role.codepipeline.arn

  artifact_store {
    location = aws_s3_bucket.artifacts.bucket
    type     = "S3"
  }

  stage {
    name = "Source"

    action {
      name             = "Source"
      category         = "Source"
      owner            = "AWS"
      provider         = "CodeStarSourceConnection"
      version          = "1"
      output_artifacts = ["source_output"]

      configuration = {
        ConnectionArn    = var.github_connection_arn
        FullRepositoryId = var.github_repository_id
        BranchName       = var.github_branch
      }
    }
  }

  stage {
    name = "Build"

    action {
      name             = "Build"
      category         = "Build"
      owner            = "AWS"
      provider         = "CodeBuild"
      input_artifacts  = ["source_output"]
      output_artifacts = ["build_output"]
      version          = "1"

      configuration = {
        ProjectName = aws_codebuild_project.main.name
      }
    }
  }

  dynamic "stage" {
    for_each = var.enable_staging ? [1] : []

    content {
      name = "Staging"

      action {
        name            = "DeployStaging"
        category        = "Deploy"
        owner           = "AWS"
        provider        = "ECS"
        input_artifacts = ["build_output"]
        version         = "1"

        configuration = {
          ClusterName = var.staging_cluster_name
          ServiceName = var.staging_service_name
        }
      }
    }
  }

  stage {
    name = "Production"

    action {
      name            = "DeployProduction"
      category        = "Deploy"
      owner           = "AWS"
      provider        = "ECS"
      input_artifacts = ["build_output"]
      version         = "1"

      configuration = {
        ClusterName = var.production_cluster_name
        ServiceName = var.production_service_name
      }
    }
  }

  tags = merge(var.tags, {
    Name = var.pipeline_name
  })
}

# IAM Role for CodePipeline
resource "aws_iam_role" "codepipeline" {
  name = "${var.pipeline_name}-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "codepipeline.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "codepipeline" {
  name = "${var.pipeline_name}-policy"
  role = aws_iam_role.codepipeline.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:GetObjectVersion",
          "s3:GetBucketVersioning",
          "s3:PutObject"
        ]
        Resource = [
          aws_s3_bucket.artifacts.arn,
          "${aws_s3_bucket.artifacts.arn}/*"
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "codebuild:BatchGetBuilds",
          "codebuild:StartBuild"
        ]
        Resource = aws_codebuild_project.main.arn
      },
      {
        Effect = "Allow"
        Action = [
          "ecs:DescribeServices",
          "ecs:DescribeTasks",
          "ecs:ListTasks",
          "ecs:UpdateService"
        ]
        Resource = "*"
      }
    ]
  })
}

# S3 bucket for artifacts
resource "aws_s3_bucket" "artifacts" {
  bucket = "${var.pipeline_name}-artifacts"

  tags = merge(var.tags, {
    Name = "${var.pipeline_name}-artifacts"
  })
}

resource "aws_s3_bucket_versioning" "artifacts" {
  bucket = aws_s3_bucket.artifacts.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

---

## 📊 **Monitoring Infrastructure**

### **CloudWatch Monitoring Setup**
```hcl
# modules/monitoring/main.tf
# CloudWatch Dashboard
resource "aws_cloudwatch_dashboard" "main" {
  dashboard_name = var.dashboard_name

  dashboard_body = jsonencode({
    widgets = [
      # ECS Service CPU Utilization
      {
        type   = "metric"
        x      = 0
        y      = 0
        width  = 12
        height = 6

        properties = {
          metrics = [
            ["AWS/ECS", "CPUUtilization", "ServiceName", var.ecs_service_name, "ClusterName", var.ecs_cluster_name]
          ]
          view    = "timeSeries"
          stacked = false
          region  = var.aws_region
          title   = "ECS CPU Utilization"
          period  = 300
        }
      },

      # ECS Service Memory Utilization
      {
        type   = "metric"
        x      = 12
        y      = 0
        width  = 12
        height = 6

        properties = {
          metrics = [
            ["AWS/ECS", "MemoryUtilization", "ServiceName", var.ecs_service_name, "ClusterName", var.ecs_cluster_name]
          ]
          view    = "timeSeries"
          stacked = false
          region  = var.aws_region
          title   = "ECS Memory Utilization"
          period  = 300
        }
      },

      # RDS CPU Utilization
      {
        type   = "metric"
        x      = 0
        y      = 6
        width  = 12
        height = 6

        properties = {
          metrics = [
            ["AWS/RDS", "CPUUtilization", "DBInstanceIdentifier", var.rds_instance_id]
          ]
          view    = "timeSeries"
          stacked = false
          region  = var.aws_region
          title   = "RDS CPU Utilization"
          period  = 300
        }
      },

      # Application Response Time
      {
        type   = "metric"
        x      = 12
        y      = 6
        width  = 12
        height = 6

        properties = {
          metrics = [
            ["AWS/ApiGateway", "Latency", "ApiName", var.api_gateway_name]
          ]
          view    = "timeSeries"
          stacked = false
          region  = var.aws_region
          title   = "API Gateway Latency"
          period  = 300
        }
      }
    ]
  })
}

# CloudWatch Alarms
resource "aws_cloudwatch_metric_alarm" "ecs_cpu_high" {
  alarm_name          = "${var.project_name}-ecs-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/ECS"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "ECS CPU utilization is high"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    ClusterName = var.ecs_cluster_name
    ServiceName = var.ecs_service_name
  }
}

resource "aws_cloudwatch_metric_alarm" "rds_cpu_high" {
  alarm_name          = "${var.project_name}-rds-cpu-high"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = "2"
  metric_name         = "CPUUtilization"
  namespace           = "AWS/RDS"
  period              = "300"
  statistic           = "Average"
  threshold           = "80"
  alarm_description   = "RDS CPU utilization is high"
  alarm_actions       = [aws_sns_topic.alerts.arn]

  dimensions = {
    DBInstanceIdentifier = var.rds_instance_id
  }
}

# SNS Topic for alerts
resource "aws_sns_topic" "alerts" {
  name = "${var.project_name}-alerts"

  tags = merge(var.tags, {
    Name = "${var.project_name}-alerts"
  })
}

resource "aws_sns_topic_subscription" "email" {
  topic_arn = aws_sns_topic.alerts.arn
  protocol  = "email"
  endpoint  = var.alert_email
}

# CloudWatch Logs
resource "aws_cloudwatch_log_group" "app" {
  name              = "/ecs/${var.project_name}"
  retention_in_days = 30

  tags = merge(var.tags, {
    Name = "${var.project_name}-logs"
  })
}

resource "aws_cloudwatch_log_metric_filter" "error_count" {
  name           = "${var.project_name}-error-count"
  pattern        = "ERROR"
  log_group_name = aws_cloudwatch_log_group.app.name

  metric_transformation {
    name      = "${var.project_name}-error-count"
    namespace = "AppMetrics"
    value     = "1"
  }
}
```

---

## 🔒 **Security Infrastructure**

### **VPC Security Groups**
```hcl
# modules/security/main.tf
# Application Load Balancer Security Group
resource "aws_security_group" "alb" {
  name_prefix = "${var.project_name}-alb-"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(var.tags, {
    Name = "${var.project_name}-alb-sg"
  })
}

# ECS Tasks Security Group
resource "aws_security_group" "ecs_tasks" {
  name_prefix = "${var.project_name}-ecs-tasks-"
  vpc_id      = var.vpc_id

  ingress {
    from_port       = var.app_port
    to_port         = var.app_port
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(var.tags, {
    Name = "${var.project_name}-ecs-tasks-sg"
  })
}

# RDS Security Group
resource "aws_security_group" "rds" {
  name_prefix = "${var.project_name}-rds-"
  vpc_id      = var.vpc_id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.ecs_tasks.id]
  }

  tags = merge(var.tags, {
    Name = "${var.project_name}-rds-sg"
  })
}

# Bastion Host Security Group
resource "aws_security_group" "bastion" {
  name_prefix = "${var.project_name}-bastion-"
  vpc_id      = var.vpc_id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = var.allowed_ssh_cidr_blocks
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(var.tags, {
    Name = "${var.project_name}-bastion-sg"
  })
}
```

### **IAM Roles and Policies**
```hcl
# modules/iam/main.tf
# ECS Task Execution Role
resource "aws_iam_role" "ecs_execution" {
  name = "${var.project_name}-ecs-execution-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ecs-tasks.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "ecs_execution" {
  role       = aws_iam_role.ecs_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}

# ECS Task Role
resource "aws_iam_role" "ecs_task" {
  name = "${var.project_name}-ecs-task-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ecs-tasks.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "ecs_task" {
  name = "${var.project_name}-ecs-task-policy"
  role = aws_iam_role.ecs_task.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ]
        Resource = "${aws_s3_bucket.app_assets.arn}/*"
      },
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "${aws_cloudwatch_log_group.app.arn}:*"
      }
    ]
  })
}

# CodeBuild Role
resource "aws_iam_role" "codebuild" {
  name = "${var.project_name}-codebuild-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "codebuild.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy" "codebuild" {
  name = "${var.project_name}-codebuild-policy"
  role = aws_iam_role.codebuild.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = [
          "arn:aws:logs:${var.aws_region}:${data.aws_caller_identity.current.account_id}:log-group:/aws/codebuild/${var.project_name}",
          "arn:aws:logs:${var.aws_region}:${data.aws_caller_identity.current.account_id}:log-group:/aws/codebuild/${var.project_name}:*"
        ]
      },
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ]
        Resource = [
          "${aws_s3_bucket.codebuild_cache.arn}",
          "${aws_s3_bucket.codebuild_cache.arn}/*"
        ]
      }
    ]
  })
}
```

---

## 🌍 **Multi-Environment Management**

### **Workspace Management**
```hcl
# workspaces.tf
terraform {
  cloud {
    organization = "your-organization"

    workspaces {
      tags = ["react-native", "infrastructure"]
    }
  }
}

# Create workspaces for different environments
locals {
  environments = ["dev", "staging", "prod"]
}

resource "tfe_workspace" "environments" {
  for_each = toset(local.environments)

  name         = "${var.project_name}-${each.key}"
  organization = var.organization
  description  = "${each.key} environment for ${var.project_name}"

  working_directory = "terraform"
  terraform_version = "~> 1.0"

  vcs_repo {
    identifier     = var.github_repository
    branch         = each.key == "prod" ? "main" : each.key
    oauth_token_id = var.oauth_token_id
  }
}
```

### **Environment-Specific Variables**
```hcl
# environments/dev.tfvars
aws_region     = "us-east-1"
environment    = "dev"
project_name   = "react-native-app"

# VPC Configuration
vpc_cidr         = "10.0.0.0/16"
public_subnets   = ["10.0.1.0/24", "10.0.2.0/24"]
private_subnets  = ["10.0.10.0/24", "10.0.11.0/24"]

# ECS Configuration
ecs_desired_count = 1
ecs_instance_type = "t3.micro"

# RDS Configuration
rds_instance_class = "db.t3.micro"
rds_allocated_storage = 20

# Domain Configuration
domain_name = "dev.myapp.com"
```

```hcl
# environments/staging.tfvars
aws_region     = "us-east-1"
environment    = "staging"
project_name   = "react-native-app"

# VPC Configuration
vpc_cidr         = "10.1.0.0/16"
public_subnets   = ["10.1.1.0/24", "10.1.2.0/24", "10.1.3.0/24"]
private_subnets  = ["10.1.10.0/24", "10.1.11.0/24", "10.1.12.0/24"]

# ECS Configuration
ecs_desired_count = 2
ecs_instance_type = "t3.small"

# RDS Configuration
rds_instance_class = "db.t3.small"
rds_allocated_storage = 50
rds_multi_az = true

# Domain Configuration
domain_name = "staging.myapp.com"
```

```hcl
# environments/prod.tfvars
aws_region     = "us-east-1"
environment    = "prod"
project_name   = "react-native-app"

# VPC Configuration
vpc_cidr         = "10.2.0.0/16"
public_subnets   = ["10.2.1.0/24", "10.2.2.0/24", "10.2.3.0/24"]
private_subnets  = ["10.2.10.0/24", "10.2.11.0/24", "10.2.12.0/24"]

# ECS Configuration
ecs_desired_count = 3
ecs_instance_type = "t3.medium"

# RDS Configuration
rds_instance_class = "db.t3.medium"
rds_allocated_storage = 100
rds_multi_az = true
rds_backup_retention = 30

# Domain Configuration
domain_name = "myapp.com"
```

### **Deployment Script**
```bash
#!/bin/bash
# deploy-terraform.sh

set -e

# Configuration
ENVIRONMENT=${1:-dev}
WORKSPACE="${ENVIRONMENT}"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log() {
  echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

error() {
  echo -e "${RED}[ERROR] $1${NC}"
}

info() {
  echo -e "${BLUE}[INFO] $1${NC}"
}

warning() {
  echo -e "${YELLOW}[WARNING] $1${NC}"
}

# Validate environment
validate_environment() {
  if [[ ! "$ENVIRONMENT" =~ ^(dev|staging|prod)$ ]]; then
    error "Invalid environment: $ENVIRONMENT"
    echo "Valid environments: dev, staging, prod"
    exit 1
  fi

  if [ ! -f "environments/${ENVIRONMENT}.tfvars" ]; then
    error "Environment configuration not found: environments/${ENVIRONMENT}.tfvars"
    exit 1
  fi
}

# Initialize Terraform
init_terraform() {
  log "Initializing Terraform..."
  terraform init -upgrade
}

# Select workspace
select_workspace() {
  log "Selecting workspace: $WORKSPACE"

  # Create workspace if it doesn't exist
  if ! terraform workspace select "$WORKSPACE" 2>/dev/null; then
    log "Creating workspace: $WORKSPACE"
    terraform workspace new "$WORKSPACE"
  fi
}

# Plan changes
plan_changes() {
  log "Planning infrastructure changes..."

  terraform plan \
    -var-file="environments/${ENVIRONMENT}.tfvars" \
    -out=tfplan \
    -detailed-exitcode

  local exit_code=$?
  if [ $exit_code -eq 2 ]; then
    warning "Infrastructure changes detected"
    return 0
  elif [ $exit_code -eq 1 ]; then
    error "Terraform plan failed"
    exit 1
  else
    info "No infrastructure changes detected"
    return 1
  fi
}

# Apply changes
apply_changes() {
  log "Applying infrastructure changes..."

  if [ "$ENVIRONMENT" = "prod" ]; then
    warning "Applying changes to PRODUCTION environment"
    read -p "Are you sure you want to continue? (yes/no): " -n 3 -r
    echo
    if [[ ! $REPLY =~ ^yes$ ]]; then
      log "Deployment cancelled"
      exit 0
    fi
  fi

  terraform apply tfplan
}

# Run validation
run_validation() {
  log "Running infrastructure validation..."

  # Check if resources are healthy
  # Add your validation logic here
  log "Infrastructure validation completed"
}

# Main deployment function
main() {
  log "Starting Terraform deployment for environment: $ENVIRONMENT"

  validate_environment
  init_terraform
  select_workspace

  if plan_changes; then
    apply_changes
    run_validation
    log "✅ Deployment completed successfully!"
  else
    log "ℹ️  No changes to apply"
  fi
}

# Run main function
main "$@"
```

---

## 💰 **Cost Optimization**

### **Cost Monitoring**
```hcl
# modules/cost/main.tf
# Cost Allocation Tags
resource "aws_ce_cost_allocation_tag" "environment" {
  tag_key = "Environment"
  status  = "Active"
}

resource "aws_ce_cost_allocation_tag" "project" {
  tag_key = "Project"
  status  = "Active"
}

_ce_cost_allocation_tag" "project" {
  tag_key = "Project"
  status  = "Active"
}

resource "aws_ce_cost_allocation_tag" "team" {
  tag_key = "Team"
  status  = "Active"
}

# Cost Budget
resource "aws_budgets_budget" "monthly" {
  name         = "${var.project_name}-monthly-budget"
  budget_type  = "COST"
  limit_amount = var.monthly_budget_limit
  limit_unit   = "USD"
  time_unit    = "MONTHLY"

  cost_filter {
    name   = "TagKeyValue"
    values = ["Project$${var.project_name}"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type            = "PERCENTAGE"
    notification_type         = "FORECASTED"
    subscriber_email_addresses = var.budget_alert_emails
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 100
    threshold_type            = "PERCENTAGE"
    notification_type         = "ACTUAL"
    subscriber_email_addresses = var.budget_alert_emails
  }
}

# Cost Anomaly Detection
resource "aws_ce_anomaly_monitor" "service" {
  name              = "${var.project_name}-anomaly-monitor"
  monitor_type      = "DIMENSIONAL"
  monitor_dimension = "SERVICE"
}

resource "aws_ce_anomaly_subscription" "alert" {
  name      = "${var.project_name}-anomaly-alert"
  threshold = 100

  frequency = "DAILY"

  monitor_arn_list = [
    aws_ce_anomaly_monitor.service.arn
  ]

  subscriber {
    type    = "EMAIL"
    address = var.cost_anomaly_email
  }
}
```

### **Auto Scaling for Cost Optimization**
```hcl
# modules/autoscaling/main.tf
# Application Auto Scaling for ECS
resource "aws_appautoscaling_target" "ecs_target" {
  max_capacity       = var.max_capacity
  min_capacity       = var.min_capacity
  resource_id        = "service/${var.ecs_cluster_name}/${var.ecs_service_name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

resource "aws_appautoscaling_policy" "ecs_cpu" {
  name               = "${var.project_name}-cpu-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs_target.scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs_target.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    target_value = 70.0
  }
}

resource "aws_appautoscaling_policy" "ecs_memory" {
  name               = "${var.project_name}-memory-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs_target.scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs_target.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageMemoryUtilization"
    }
    target_value = 80.0
  }
}

# Scheduled scaling for cost optimization
resource "aws_appautoscaling_scheduled_action" "scale_down" {
  name               = "${var.project_name}-scale-down"
  service_namespace  = aws_appautoscaling_target.ecs_target.service_namespace
  resource_id        = aws_appautoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs_target.scalable_dimension
  schedule           = "cron(0 18 * * ? *)" # 6 PM daily

  scalable_target_action {
    min_capacity = 1
    max_capacity = 2
  }
}

resource "aws_appautoscaling_scheduled_action" "scale_up" {
  name               = "${var.project_name}-scale-up"
  service_namespace  = aws_appautoscaling_target.ecs_target.service_namespace
  resource_id        = aws_appautoscaling_target.ecs_target.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs_target.scalable_dimension
  schedule           = "cron(0 6 * * ? *)" # 6 AM daily

  scalable_target_action {
    min_capacity = var.min_capacity
    max_capacity = var.max_capacity
  }
}
```

---

## 📱 **Practical Examples**

### **Complete Infrastructure Setup**
```hcl
# main.tf - Complete infrastructure setup
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }

  backend "s3" {
    bucket = "react-native-infra-state"
    key    = "terraform/state"
    region = "us-east-1"
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
    }
  }
}

# VPC Module
module "vpc" {
  source = "./modules/vpc"

  name             = "${var.project_name}-vpc"
  cidr_block       = var.vpc_cidr
  public_subnets   = var.public_subnets
  private_subnets  = var.private_subnets
  availability_zones = var.availability_zones
  enable_nat_gateway = var.environment != "dev"

  tags = {
    Environment = var.environment
    Project     = var.project_name
  }
}

# Security Module
module "security" {
  source = "./modules/security"

  project_name           = var.project_name
  vpc_id                 = module.vpc.vpc_id
  app_port              = var.app_port
  allowed_ssh_cidr_blocks = var.allowed_ssh_cidr_blocks

  tags = {
    Environment = var.environment
  }
}

# RDS Database
module "rds" {
  source = "./modules/rds"

  identifier              = "${var.project_name}-db"
  engine                 = "postgres"
  engine_version         = "14.7"
  instance_class         = var.rds_instance_class
  allocated_storage      = var.rds_allocated_storage
  max_allocated_storage  = var.rds_max_allocated_storage
  db_name                = var.db_name
  username               = var.db_username
  password               = var.db_password
  vpc_security_group_ids = [module.security.rds_sg_id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  backup_retention_period = var.backup_retention_period
  multi_az               = var.environment == "prod"
  skip_final_snapshot    = var.environment != "prod"

  tags = {
    Environment = var.environment
  }
}

# ECS Cluster
module "ecs" {
  source = "./modules/ecs"

  cluster_name = "${var.project_name}-cluster"
  service_name = "${var.project_name}-service"

  task_definition = {
    family                   = "${var.project_name}-task"
    network_mode            = "awsvpc"
    requires_compatibilities = ["FARGATE"]
    cpu                     = var.task_cpu
    memory                  = var.task_memory
    execution_role_arn      = aws_iam_role.ecs_execution.arn
    task_role_arn          = aws_iam_role.ecs_task.arn

    container_definitions = jsonencode([
      {
        name  = "app"
        image = "${aws_ecr_repository.app.repository_url}:latest"
        portMappings = [
          {
            containerPort = var.app_port
            hostPort      = var.app_port
            protocol      = "tcp"
          }
        ]
        environment = [
          {
            name  = "NODE_ENV"
            value = var.environment
          },
          {
            name  = "DATABASE_URL"
            value = "postgresql://${var.db_username}:${var.db_password}@${module.rds.endpoint}/${var.db_name}"
          }
        ]
        logConfiguration = {
          logDriver = "awslogs"
          options = {
            "awslogs-group"         = aws_cloudwatch_log_group.app.name
            "awslogs-region"        = var.aws_region
            "awslogs-stream-prefix" = "ecs"
          }
        }
      }
    ])
  }

  desired_count = var.ecs_desired_count
  subnet_ids    = module.vpc.private_subnet_ids
  security_groups = [module.security.ecs_tasks_sg_id]

  load_balancer = {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = var.app_port
  }

  tags = {
    Environment = var.environment
  }
}

# Application Load Balancer
resource "aws_lb" "app" {
  name               = "${var.project_name}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [module.security.alb_sg_id]
  subnets            = module.vpc.public_subnet_ids

  enable_deletion_protection = var.environment == "prod"

  tags = {
    Name        = "${var.project_name}-alb"
    Environment = var.environment
  }
}

resource "aws_lb_target_group" "app" {
  name        = "${var.project_name}-tg"
  port        = var.app_port
  protocol    = "HTTP"
  vpc_id      = module.vpc.vpc_id
  target_type = "ip"

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 2
  }

  tags = {
    Name        = "${var.project_name}-tg"
    Environment = var.environment
  }
}

resource "aws_lb_listener" "app" {
  load_balancer_arn = aws_lb.app.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app.arn
  }
}

# ECR Repository
resource "aws_ecr_repository" "app" {
  name                 = var.project_name
  image_tag_mutability = "MUTABLE"

  image_scanning_configuration {
    scan_on_push = true
  }

  tags = {
    Name        = var.project_name
    Environment = var.environment
  }
}

# CloudWatch Logs
resource "aws_cloudwatch_log_group" "app" {
  name              = "/ecs/${var.project_name}"
  retention_in_days = 30

  tags = {
    Name        = "${var.project_name}-logs"
    Environment = var.environment
  }
}

# IAM Roles
resource "aws_iam_role" "ecs_execution" {
  name = "${var.project_name}-ecs-execution-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ecs-tasks.amazonaws.com"
        }
      }
    ]
  })
}

resource "aws_iam_role_policy_attachment" "ecs_execution" {
  role       = aws_iam_role.ecs_execution.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy"
}

resource "aws_iam_role" "ecs_task" {
  name = "${var.project_name}-ecs-task-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "ecs-tasks.amazonaws.com"
        }
      }
    ]
  })
}

# DB Subnet Group
resource "aws_db_subnet_group" "main" {
  name       = "${var.project_name}-db-subnet-group"
  subnet_ids = module.vpc.private_subnet_ids

  tags = {
    Name        = "${var.project_name}-db-subnet-group"
    Environment = var.environment
  }
}
```

### **Infrastructure Testing**
```hcl
# test/main.tf - Infrastructure testing
terraform {
  required_providers {
    test = {
      source = "terraform.io/builtin/test"
    }
  }
}

# Test VPC creation
run "test_vpc_creation" {
  command = plan

  variables {
    project_name = "test-app"
    environment  = "test"
    vpc_cidr     = "10.0.0.0/16"
  }

  assert {
    condition     = aws_vpc.main.cidr_block == "10.0.0.0/16"
    error_message = "VPC CIDR block should be 10.0.0.0/16"
  }

  assert {
    condition     = length(aws_subnet.public) == 2
    error_message = "Should have 2 public subnets"
  }

  assert {
    condition     = length(aws_subnet.private) == 2
    error_message = "Should have 2 private subnets"
  }
}

# Test security group rules
run "test_security_groups" {
  command = plan

  variables {
    project_name = "test-app"
    environment  = "test"
  }

  assert {
    condition     = aws_security_group.alb.ingress[0].from_port == 80
    error_message = "ALB should allow HTTP traffic on port 80"
  }

  assert {
    condition     = aws_security_group.ecs_tasks.ingress[0].from_port == var.app_port
    error_message = "ECS tasks should allow traffic on app port"
  }
}

# Test RDS configuration
run "test_rds_configuration" {
  command = plan

  variables {
    project_name = "test-app"
    environment  = "test"
  }

  assert {
    condition     = aws_db_instance.main.engine == "postgres"
    error_message = "RDS should use PostgreSQL engine"
  }

  assert {
    condition     = aws_db_instance.main.multi_az == false
    error_message = "Test environment should not use Multi-AZ"
  }
}
```

### **Terraform Registry Module**
```hcl
# Using Terraform Registry modules
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 3.0"

  name = "${var.project_name}-vpc"
  cidr = var.vpc_cidr

  azs             = var.availability_zones
  public_subnets  = var.public_subnets
  private_subnets = var.private_subnets

  enable_nat_gateway = var.environment != "dev"
  single_nat_gateway = var.environment == "dev"

  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Environment = var.environment
    Project     = var.project_name
    Terraform   = "true"
  }
}

module "ecs" {
  source  = "terraform-aws-modules/ecs/aws"
  version = "~> 4.0"

  cluster_name = "${var.project_name}-cluster"

  cluster_configuration = {
    execute_command_configuration = {
      logging = "OVERRIDE"
      log_configuration = {
        cloud_watch_log_group_name = aws_cloudwatch_log_group.ecs.name
      }
    }
  }

  fargate_capacity_providers = {
    FARGATE = {
      default_capacity_provider_strategy = {
        weight = 100
      }
    }
  }

  tags = {
    Environment = var.environment
    Project     = var.project_name
  }
}

module "alb" {
  source  = "terraform-aws-modules/alb/aws"
  version = "~> 8.0"

  name = "${var.project_name}-alb"

  load_balancer_type = "application"

  vpc_id          = module.vpc.vpc_id
  subnets         = module.vpc.public_subnets
  security_groups = [module.security.alb_sg_id]

  target_groups = [
    {
      name             = "${var.project_name}-tg"
      backend_protocol = "HTTP"
      backend_port     = var.app_port
      target_type      = "ip"

      health_check = {
        enabled             = true
        interval            = 30
        path                = "/health"
        port                = "traffic-port"
        healthy_threshold   = 2
        unhealthy_threshold = 2
        timeout             = 5
        protocol            = "HTTP"
        matcher             = "200"
      }
    }
  ]

  http_tcp_listeners = [
    {
      port               = 80
      protocol           = "HTTP"
      target_group_index = 0
    }
  ]

  tags = {
    Environment = var.environment
    Project     = var.project_name
  }
}
```

---

## 📚 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **IaC Fundamentals**: Infrastructure as Code principles and benefits
- ✅ **Terraform Basics**: Installation, configuration, and core commands
- ✅ **AWS Infrastructure**: VPC, EC2, RDS, and networking setup
- ✅ **Database Infrastructure**: PostgreSQL and DynamoDB configuration
- ✅ **CI/CD Infrastructure**: CodeBuild and CodePipeline setup
- ✅ **Monitoring Infrastructure**: CloudWatch dashboards and alerts
- ✅ **Security Infrastructure**: Security groups, IAM roles, and policies
- ✅ **Multi-Environment Management**: Workspace and environment management
- ✅ **Cost Optimization**: Budget monitoring and auto-scaling
- ✅ **Practical Examples**: Complete infrastructure setup and testing

### **Best Practices**
1. **Use modules**: Break infrastructure into reusable modules
2. **Version control**: Keep Terraform code in Git with proper branching
3. **State management**: Use remote state with locking
4. **Validate changes**: Always run `terraform plan` before applying
5. **Use variables**: Parameterize configurations for different environments
6. **Tag resources**: Implement consistent tagging for cost tracking
7. **Security first**: Apply least privilege principles
8. **Monitor costs**: Set up budgets and alerts
9. **Test infrastructure**: Use Terratest or similar tools
10. **Document changes**: Keep infrastructure changes well-documented

### **Next Steps**
- Learn advanced Terraform features (workspaces, modules)
- Implement Kubernetes with Terraform
- Set up multi-cloud infrastructure
- Learn infrastructure testing with Terratest
- Explore Terraform Cloud and Enterprise features

---

## 🎯 **Assignment**

### **Task 1: Basic Infrastructure Setup**
Create a Terraform configuration that:
- Sets up a VPC with public and private subnets
- Creates security groups for web and database access
- Provisions an EC2 instance with proper security
- Configures an S3 bucket for static assets
- Uses proper tagging and naming conventions

### **Task 2: Database Infrastructure**
Extend the infrastructure to include:
- RDS PostgreSQL instance with proper security
- Database subnet group in private subnets
- Backup and maintenance window configuration
- Read replica for high availability
- Connection from application servers only

### **Task 3: Complete Application Stack**
Implement a full application infrastructure that:
- Uses ECS Fargate for containerized applications
- Sets up Application Load Balancer with proper routing
- Configures CloudWatch monitoring and alerts
- Implements auto-scaling based on CPU/memory usage
- Sets up CI/CD pipeline with CodePipeline
- Includes cost monitoring and optimization

---

## 📚 **Additional Resources**
- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS Provider Documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Registry](https://registry.terraform.io/)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Terratest](https://terratest.gruntwork.io/)
- [Terraform Cloud](https://cloud.hashicorp.com/products/terraform)

---

*This lesson provides comprehensive coverage of Infrastructure as Code using Terraform, with practical examples for deploying React Native applications on AWS.*
resource "aws