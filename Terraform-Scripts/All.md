# 1. S3 Create
```
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}

resource "aws_s3_bucket" "my_bucket" {
  bucket = "anil-terraform-practice-2026"

  tags = {
    Name        = "Anil Terraform Bucket"
    Environment = "Dev"
  }
}
resource "aws_s3_object" "my_object" {
    bucket  = aws_s3_bucket.my_bucket.id
    key     = "devops.txt"
    content = "Hii Im Anil Kumar and leanring DevOps with AWS Terraform"
}
```

# 2. IAM User and Policy and Permission Boundary with output block
```
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

# IAM User
resource "aws_iam_user" "s3_user" {
  name = "anil-s3-user"

  permissions_boundary = "arn:aws:iam::aws:policy/AdministratorAccess"

  tags = {
    Name        = "Anil S3 User"
    Environment = "Dev"
  }
}

# S3 Policy
resource "aws_iam_policy" "s3_policy" {
  name        = "AnilS3OnlyPolicy"
  description = "Allow access only to anil-terraform-practice-2026 S3 bucket"

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [
      {
        Effect = "Allow"

        Action = [
          "s3:ListBucket"
        ]

        Resource = "arn:aws:s3:::anil-terraform-practice-2026"
      },
      {
        Effect = "Allow"

        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ]

        Resource = "arn:aws:s3:::anil-terraform-practice-2026/*"
      }
    ]
  })
}

# Attach S3 Policy to IAM User
resource "aws_iam_user_policy_attachment" "s3_policy_attachment" {
  user       = aws_iam_user.s3_user.name
  policy_arn = aws_iam_policy.s3_policy.arn
}
# Output: IAM User Name
output "iam_user_name" {
  description = "Name of the created IAM user"
  value       = aws_iam_user.s3_user.name
}

# Output: IAM User ARN
output "iam_user_arn" {
  description = "ARN of the created IAM user"
  value       = aws_iam_user.s3_user.arn
}

# Output: IAM User Unique ID
output "iam_user_unique_id" {
  description = "Unique ID of the IAM user"
  value       = aws_iam_user.s3_user.unique_id
}

# Output: S3 Policy ARN
output "s3_policy_arn" {
  description = "ARN of the custom S3 policy"
  value       = aws_iam_policy.s3_policy.arn
}

# Output: Attached Policy ARN
output "attached_s3_policy_arn" {
  description = "ARN of the policy attached to the IAM user"
  value       = aws_iam_user_policy_attachment.s3_policy_attachment.policy_arn
}
```
# 3. EC2 Instance
```
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.100.0"
    }
  }
}

provider "aws" {
 alias = "mumbai"
 region = "ap-south-1"
}

resource "aws_instance" "dev" {
    provider = aws.mumbai
    vpc_security_group_ids = [aws_security_group.main-sg.id ]
    ami = "ami-0038df39db13a87e2"
    instance_type = "t2.small"
    tags = {
      Name = "Terraform-Web-Server"
    }
}

resource "aws_security_group" "main-sg" {
    name = "dev-server-sg"
    description = "It will allow all traffic"
    ingress {
        from_port = 0
        to_port = 0
        protocol = "-1"
        cidr_blocks = ["0.0.0.0/0"]
    }
    egress {
        from_port = 0
        to_port = 0
        protocol = "-1"
        cidr_blocks = ["0.0.0.0/0"]

    }
    tags = {
      Name = "test-sgs"
    }  
}
# Output: EC2 Instance ID
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.dev.id
}

# Output: EC2 Public IP
output "public_ip" {
  description = "Public IP address of the EC2 instance"
  value       = aws_instance.dev.public_ip
}

# Output: EC2 Private IP
output "private_ip" {
  description = "Private IP address of the EC2 instance"
  value       = aws_instance.dev.private_ip
}

# Output: EC2 Instance Type
output "instance_type" {
  description = "Type of the EC2 instance"
  value       = aws_instance.dev.instance_type
}

# Output: Availability Zone
output "availability_zone" {
  description = "Availability Zone of the EC2 instance"
  value       = aws_instance.dev.availability_zone
}

# Output: AMI ID
output "ami_id" {
  description = "AMI ID used for the EC2 instance"
  value       = aws_instance.dev.ami
}

# Output: Security Group ID
output "security_group_id" {
  description = "ID of the Security Group"
  value       = aws_security_group.main-sg.id
}

# Output: Security Group Name
output "security_group_name" {
  description = "Name of the Security Group"
  value       = aws_security_group.main-sg.name
}
```

















