# Terraform Structure

terraform-VPC-EC2-Airtel-Project/
```
├── terraform.tf
├── provider.tf
├── variables.tf
├── main.tf
└── terraform.tfvars
```

# 1. terraform.tf
```

terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}
```
# 2. provider.tf
```

provider "aws" {
  region = var.aws_region
}
```
# 3. variables.tf — with validation functions
```

variable "aws_region" {
  description = "AWS deployment region"
  type        = string
  default     = "ap-south-1"

  validation {
    condition     = contains(["ap-south-1", "ap-south-2", "us-east-1", "us-west-2"], var.aws_region)
    error_message = "Choose a supported region from the configured list."
  }
}

variable "ami_id" {
  description = "AMI ID for EC2 instances"
  type        = string

  validation {
    condition     = can(regex("^ami-[0-9a-fA-F]+$", var.ami_id))
    error_message = "AMI ID must start with ami- followed by hexadecimal characters."
  }
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"

  validation {
    condition     = contains(["t2.micro", "t3.micro", "t3.small", "t3.medium"], var.instance_type)
    error_message = "Choose t2.micro, t3.micro, t3.small, or t3.medium."
  }
}

variable "key_name" {
  description = "Existing EC2 key pair name"
  type        = string

  validation {
    condition     = length(trimspace(var.key_name)) > 0
    error_message = "The EC2 key pair name cannot be empty."
  }
}

variable "ssh_cidr" {
  description = "Your public IP address in CIDR notation"
  type        = string

  validation {
    condition     = can(cidrnetmask(var.ssh_cidr))
    error_message = "Provide a valid CIDR, for example 198.51.100.10/32."
  }

  validation {
    condition     = can(cidrnetmask(var.ssh_cidr)) ? tonumber(split("/", var.ssh_cidr)[1]) == 32 : false
    error_message = "SSH access must use a single IPv4 address with /32."
  }
}
```
# 3. main.tf with output block
```

# =========================================================
# VPC
# =========================================================

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "anil-vpc"
  }
}

# =========================================================
# PUBLIC SUBNETS
# =========================================================

resource "aws_subnet" "public_a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "${var.aws_region}a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-a"
  }
}

resource "aws_subnet" "public_b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  availability_zone       = "${var.aws_region}b"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-b"
  }
}

# =========================================================
# PRIVATE SUBNETS
# =========================================================

resource "aws_subnet" "private_a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.3.0/24"
  availability_zone = "${var.aws_region}a"

  tags = {
    Name = "private-subnet-a"
  }
}

resource "aws_subnet" "private_b" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.4.0/24"
  availability_zone = "${var.aws_region}b"

  tags = {
    Name = "private-subnet-b"
  }
}

# =========================================================
# INTERNET GATEWAY
# =========================================================

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "anil-igw"
  }
}

# =========================================================
# PUBLIC ROUTE TABLE
# =========================================================

resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "public-route-table"
  }
}

resource "aws_route_table_association" "public_a" {
  subnet_id      = aws_subnet.public_a.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_route_table_association" "public_b" {
  subnet_id      = aws_subnet.public_b.id
  route_table_id = aws_route_table.public_rt.id
}

# =========================================================
# NAT GATEWAY AND ELASTIC IP
# =========================================================

resource "aws_eip" "nat_eip" {
  domain = "vpc"

  tags = {
    Name = "anil-nat-eip"
  }
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public_b.id

  depends_on = [
    aws_internet_gateway.igw
  ]

  tags = {
    Name = "anil-nat-gateway"
  }
}

# =========================================================
# PRIVATE ROUTE TABLE
# =========================================================

resource "aws_route_table" "private_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags = {
    Name = "private-route-table"
  }
}

resource "aws_route_table_association" "private_a" {
  subnet_id      = aws_subnet.private_a.id
  route_table_id = aws_route_table.private_rt.id
}

resource "aws_route_table_association" "private_b" {
  subnet_id      = aws_subnet.private_b.id
  route_table_id = aws_route_table.private_rt.id
}

# =========================================================
# PUBLIC SECURITY GROUP
# =========================================================

resource "aws_security_group" "public_sg" {
  name        = "anil-public-sg"
  description = "Allow SSH from my IP and HTTP from internet"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "SSH from my public IP"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.ssh_cidr]
  }

  ingress {
    description = "HTTP web traffic"
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
    Name = "anil-public-sg"
  }
}

# =========================================================
# PRIVATE SECURITY GROUP
# =========================================================

resource "aws_security_group" "private_sg" {
  name        = "anil-private-sg"
  description = "Allow SSH from public EC2 security group"
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
    Name = "anil-private-sg"
  }
}

# =========================================================
# PUBLIC EC2 - APACHE WEB SERVER
# =========================================================

resource "aws_instance" "public_ec2" {
  ami                         = var.ami_id
  instance_type               = var.instance_type
  key_name                    = var.key_name
  subnet_id                   = aws_subnet.public_b.id
  vpc_security_group_ids      = [aws_security_group.public_sg.id]
  associate_public_ip_address = true

  user_data = <<-EOF
    #!/bin/bash
    dnf install -y httpd
    systemctl enable --now httpd
    echo "Hello from Terraform Public EC2" > /var/www/html/index.html
  EOF

  depends_on = [
    aws_route_table_association.public_b
  ]

  tags = {
    Name = "Public-Web-Server"
  }
}

# =========================================================
# PRIVATE EC2
# =========================================================

resource "aws_instance" "private_ec2" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  key_name               = var.key_name
  subnet_id              = aws_subnet.private_b.id
  vpc_security_group_ids = [aws_security_group.private_sg.id]

  depends_on = [
    aws_route_table_association.private_b
  ]

  tags = {
    Name = "Private-Server"
  }
}

# =========================================================
# OUTPUTS
# =========================================================

output "vpc_id" {
  description = "Created VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = [aws_subnet.public_a.id, aws_subnet.public_b.id]
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = [aws_subnet.private_a.id, aws_subnet.private_b.id]
}

output "internet_gateway_id" {
  description = "Internet Gateway ID"
  value       = aws_internet_gateway.igw.id
}

output "nat_gateway_id" {
  description = "NAT Gateway ID"
  value       = aws_nat_gateway.nat.id
}

output "public_instance_id" {
  description = "Public EC2 instance ID"
  value       = aws_instance.public_ec2.id
}

output "public_instance_ip" {
  description = "Public EC2 public IP address"
  value       = aws_instance.public_ec2.public_ip
}

output "private_instance_id" {
  description = "Private EC2 instance ID"
  value       = aws_instance.private_ec2.id
}

output "private_instance_ip" {
  description = "Private EC2 private IP address"
  value       = aws_instance.private_ec2.private_ip
}

output "website_url" {
  description = "Apache web server URL"
  value       = "http://${aws_instance.public_ec2.public_ip}"
}

```

# 4. terraform.tfvars
```

aws_region    = "ap-south-1"
ami_id        = "ami-0011550b539717e2a"
instance_type = "t3.micro"
key_name      = "newanil6969d"
ssh_cidr      = "YOUR_CURRENT_PUBLIC_IP/32"
```
