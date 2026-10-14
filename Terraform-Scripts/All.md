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

# 4. AWS resource	Quantity
VPC	1
Public subnets	2
Private subnets	2
Internet Gateway	1
NAT Gateway	1
Elastic IP	1
Public route table	1
Private route table	1
Route table associations	4
Security groups	2
Public EC2	1
Private EC2 2 with output block code
# main.tf
```

terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "5.100.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}

# Variables
variable "ami_id" {
  description = "Valid x86_64 AMI ID in ap-south-1"
  type        = string
}

variable "key_name" {
  description = "Existing EC2 key pair name in ap-south-1"
  type        = string
}

variable "ssh_cidr" {
  description = "Your public IPv4 address with /32"
  type        = string
}

# 1. VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "Anil-VPC"
  }
}

# 2. Public Subnet A
resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "Public-Subnet-A"
  }
}

# 3. Public Subnet B
resource "aws_subnet" "public_b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "ap-south-1b"
  map_public_ip_on_launch = true

  tags = {
    Name = "Public-Subnet-B"
  }
}

# 4. Private Subnet A
resource "aws_subnet" "private_a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.3.0/24"
  availability_zone = "ap-south-1a"

  tags = {
    Name = "Private-Subnet-A"
  }
}

# 5. Private Subnet B
resource "aws_subnet" "private_b" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.4.0/24"
  availability_zone = "ap-south-1b"

  tags = {
    Name = "Private-Subnet-B"
  }
}

# 6. Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "Anil-IGW"
  }
}

# 7. Public Route Table
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "Public-Route-Table"
  }
}

# 8. Public Route Table Associations
resource "aws_route_table_association" "public_a" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_route_table_association" "public_b" {
  subnet_id      = aws_subnet.public_b.id
  route_table_id = aws_route_table.public_rt.id
}

# 9. Elastic IP for NAT Gateway
resource "aws_eip" "nat_eip" {
  domain = "vpc"

  tags = {
    Name = "Anil-NAT-EIP"
  }

  depends_on = [aws_internet_gateway.igw]
}

# 10. NAT Gateway in AZ A
resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public_a.id

  tags = {
    Name = "Anil-NAT-Gateway"
  }

  depends_on = [aws_internet_gateway.igw]
}

# 11. Private Route Table
resource "aws_route_table" "private_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags = {
    Name = "Private-Route-Table"
  }
}

# 12. Private Route Table Associations
resource "aws_route_table_association" "private_a" {
  subnet_id      = aws_subnet.private_a.id
  route_table_id = aws_route_table.private_rt.id
}

resource "aws_route_table_association" "private_b" {
  subnet_id      = aws_subnet.private_b.id
  route_table_id = aws_route_table.private_rt.id
}

# 13. Public EC2 Security Group
resource "aws_security_group" "public_sg" {
  name        = "anil-public-sg"
  description = "Allow HTTP and restricted SSH"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "SSH from trusted IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.ssh_cidr]
  }

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "Public-EC2-SG"
  }
}

# 14. Private EC2 Security Group
resource "aws_security_group" "private_sg" {
  name        = "anil-private-sg"
  description = "Allow SSH from public EC2"
  vpc_id      = aws_vpc.main.id

  ingress {
    description     = "SSH from public EC2"
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = [aws_security_group.public_sg.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "Private-EC2-SG"
  }
}

# 15. Public EC2 in AZ B
resource "aws_instance" "public_ec2" {
  ami                         = var.ami_id
  instance_type               = "t2.micro"
  key_name                    = var.key_name
  subnet_id                   = aws_subnet.public_b.id
  vpc_security_group_ids      = [aws_security_group.public_sg.id]
  associate_public_ip_address = true

  user_data = <<-EOF
    #!/bin/bash
    dnf install -y httpd
    systemctl enable --now httpd
    echo "<h1>Anil Terraform Web Server</h1>" > /var/www/html/index.html
  EOF

  tags = {
    Name = "Public-Web-Server"
  }

  depends_on = [
    aws_route_table_association.public_b
  ]
}

# 16. Private EC2 in AZ B
resource "aws_instance" "private_ec2" {
  ami                    = var.ami_id
  instance_type          = "t2.micro"
  key_name               = var.key_name
  subnet_id              = aws_subnet.private_b.id
  vpc_security_group_ids = [aws_security_group.private_sg.id]

  tags = {
    Name = "Private-Server"
  }

  depends_on = [
    aws_route_table_association.private_b
  ]
}

# 17. Outputs
output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_a_id" {
  value = aws_subnet.public_a.id
}

output "public_subnet_b_id" {
  value = aws_subnet.public_b.id
}

output "private_subnet_a_id" {
  value = aws_subnet.private_a.id
}

output "private_subnet_b_id" {
  value = aws_subnet.private_b.id
}

output "internet_gateway_id" {
  value = aws_internet_gateway.igw.id
}

output "nat_gateway_id" {
  value = aws_nat_gateway.nat.id
}

output "nat_gateway_public_ip" {
  value = aws_eip.nat_eip.public_ip
}

output "public_route_table_id" {
  value = aws_route_table.public_rt.id
}

output "private_route_table_id" {
  value = aws_route_table.private_rt.id
}

output "public_ec2_id" {
  value = aws_instance.public_ec2.id
}

output "public_ec2_ip" {
  value = aws_instance.public_ec2.public_ip
}

output "private_ec2_id" {
  value = aws_instance.private_ec2.id
}

output "private_ec2_ip" {
  value = aws_instance.private_ec2.private_ip
}

output "web_server_url" {
  value = "http://${aws_instance.public_ec2.public_ip}"
}
```

# terraform.tfvars

ami_id   = "ami-0011550b539717e2a"
key_name = "newanil6969d"
ssh_cidr = "106.200.27.215/32"

















