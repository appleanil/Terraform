Anil, let's build a complete AWS VPC infrastructure using Terraform modules. This project will create a VPC, public and private subnets, Internet Gateway (IGW), NAT Gateway, route tables, and EC2 instances.

We'll use your preferred AWS region, `ap-south-1` (Mumbai), and separate reusable modules for each major component.

## 1. Architecture

Internet

Internet Gateway (IGW)

## VPC — 10.0.0.0/16

Public subnet

10.0.1.0/24

EC2 + NAT Gateway

Internet route

Private subnet

10.0.2.0/24

Private EC2

Outbound via NAT

Public route table → IGW · Private route table → NAT Gateway

How it works:

- Public EC2: can access the internet when assigned a public IP and permitted by its security group.
- Private EC2: has no public IP and uses the NAT Gateway for outbound internet access.
- NAT Gateway: must be placed in a public subnet and uses an Elastic IP.

Note: NAT Gateways incur charges while provisioned, even when lightly used.

## 2. Project folder structure

Create this folder structure in your Terraform project.

```

terraform-vpc-project/
├── provider.tf
├── main.tf
├── variables.tf
├── terraform.tfvars
├── outputs.tf
│
└── modules/
    ├── vpc/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    └── ec2/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf

```

## 3. Root module files

### `provider.tf`

```

terraform {
  required_version = ">= 1.10.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

```

### `main.tf`

This calls both reusable modules and connects the EC2 instances to the correct subnets.

```

module "vpc" {
  source = "./modules/vpc"

  vpc_name   = var.vpc_name
  vpc_cidr   = var.vpc_cidr
  az_1       = var.az_1
  az_2       = var.az_2

  public_subnet_1a_cidr  = var.public_subnet_1a_cidr
  public_subnet_1b_cidr  = var.public_subnet_1b_cidr
  private_subnet_1a_cidr = var.private_subnet_1a_cidr
  private_subnet_1b_cidr = var.private_subnet_1b_cidr
}

module "ec2" {
  source = "./modules/ec2"

  vpc_id            = module.vpc.vpc_id
  public_subnet_id  = module.vpc.public_subnet_1a_id
  private_subnet_id = module.vpc.private_subnet_1a_id

  ami_id           = var.ami_id
  instance_type    = var.instance_type
  key_name         = var.key_name
  ssh_allowed_cidr = var.ssh_allowed_cidr
}

```

### `variables.tf`

```

variable "aws_region" {
  type    = string
  default = "ap-south-1"
}

variable "vpc_name" {
  type    = string
  default = "anil-vpc"
}

variable "vpc_cidr" {
  type    = string
  default = "10.0.0.0/16"
}

variable "az_1" {
  type    = string
  default = "ap-south-1a"
}

variable "az_2" {
  type    = string
  default = "ap-south-1b"
}

variable "public_subnet_1a_cidr" {
  type    = string
  default = "10.0.1.0/24"
}

variable "public_subnet_1b_cidr" {
  type    = string
  default = "10.0.3.0/24"
}

variable "private_subnet_1a_cidr" {
  type    = string
  default = "10.0.2.0/24"
}

variable "private_subnet_1b_cidr" {
  type    = string
  default = "10.0.4.0/24"
}

variable "ami_id" {
  description = "Valid AMI ID for the selected AWS region"
  type        = string
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

variable "key_name" {
  description = "Existing EC2 key pair name"
  type        = string
}

variable "ssh_allowed_cidr" {
  description = "Your public IP address in CIDR /32 format"
  type        = string
}

```

### `terraform.tfvars`

```

aws_region      = "ap-south-1"
vpc_name        = "anil-vpc"
ami_id          = "ami-0011550b539717e2a"
instance_type   = "t3.micro"
key_name        = "newanil6969d"

# Replace with your current public IP address followed by /32
ssh_allowed_cidr = "106.200.27.215/32"

```

The AMI must exist in `ap-south-1` and be compatible with your selected instance type. The key pair must already exist in that region.

## 4. VPC module

### `modules/vpc/variables.tf`

```

variable "vpc_name" { type = string }
variable "vpc_cidr" { type = string }

variable "az_1" { type = string }
variable "az_2" { type = string }

variable "public_subnet_1a_cidr" { type = string }
variable "public_subnet_1b_cidr" { type = string }
variable "private_subnet_1a_cidr" { type = string }
variable "private_subnet_1b_cidr" { type = string }

```

### `modules/vpc/main.tf`

This creates the VPC, four subnets, IGW, NAT Gateway, Elastic IP, and both route tables.

```

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = var.vpc_name
  }
}

resource "aws_subnet" "public_1a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_1a_cidr
  availability_zone       = var.az_1
  map_public_ip_on_launch = true

  tags = { Name = "${var.vpc_name}-public-1a" }
}

resource "aws_subnet" "public_1b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.public_subnet_1b_cidr
  availability_zone       = var.az_2
  map_public_ip_on_launch = true

  tags = { Name = "${var.vpc_name}-public-1b" }
}

resource "aws_subnet" "private_1a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_1a_cidr
  availability_zone = var.az_1

  tags = { Name = "${var.vpc_name}-private-1a" }
}

resource "aws_subnet" "private_1b" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_1b_cidr
  availability_zone = var.az_2

  tags = { Name = "${var.vpc_name}-private-1b" }
}

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = { Name = "${var.vpc_name}-igw" }
}

resource "aws_eip" "nat" {
  domain = "vpc"

  tags = { Name = "${var.vpc_name}-nat-eip" }
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public_1a.id

  depends_on = [aws_internet_gateway.igw]

  tags = { Name = "${var.vpc_name}-nat" }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = { Name = "${var.vpc_name}-public-rt" }
}

resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags = { Name = "${var.vpc_name}-private-rt" }
}

resource "aws_route_table_association" "public_1a" {
  subnet_id      = aws_subnet.public_1a.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public_1b" {
  subnet_id      = aws_subnet.public_1b.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private_1a" {
  subnet_id      = aws_subnet.private_1a.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private_1b" {
  subnet_id      = aws_subnet.private_1b.id
  route_table_id = aws_route_table.private.id
}

```

### `modules/vpc/outputs.tf`

```


output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_1a_id" {
  value = aws_subnet.public_1a.id
}

output "public_subnet_1b_id" {
  value = aws_subnet.public_1b.id
}

output "private_subnet_1a_id" {
  value = aws_subnet.private_1a.id
}

output "private_subnet_1b_id" {
  value = aws_subnet.private_1b.id
}

output "nat_gateway_id" {
  value = aws_nat_gateway.nat.id
}

output "internet_gateway_id" {
  value = aws_internet_gateway.igw.id
}


```

## 5. EC2 module

### `modules/ec2/variables.tf`

```

variable "vpc_id" { type = string }

variable "public_subnet_id" {
  type = string
}

variable "private_subnet_id" {
  type = string
}

variable "ami_id" { type = string }

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

variable "key_name" { type = string }

variable "ssh_allowed_cidr" {
  type = string
}

```

### `modules/ec2/main.tf`

This creates a public EC2 and a private EC2, with separate security groups.

```

resource "aws_security_group" "public_ec2" {
  name        = "anil-public-ec2-sg"
  description = "SSH access to public EC2"
  vpc_id      = var.vpc_id

  ingress {
    description = "SSH from trusted IP only"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.ssh_allowed_cidr]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "private_ec2" {
  name        = "anil-private-ec2-sg"
  description = "Outbound access for private EC2"
  vpc_id      = var.vpc_id

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "public" {
  ami                         = var.ami_id
  instance_type               = var.instance_type
  subnet_id                   = var.public_subnet_id
  vpc_security_group_ids      = [aws_security_group.public_ec2.id]
  key_name                    = var.key_name
  associate_public_ip_address = true

  tags = {
    Name = "anil-public-server"
  }
}

resource "aws_instance" "private" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.private_subnet_id
  vpc_security_group_ids = [aws_security_group.private_ec2.id]
  key_name               = var.key_name

  associate_public_ip_address = false

  tags = {
    Name = "anil-private-server"
  }
}

```

### `modules/ec2/outputs.tf`

```

output "public_instance_id" {
  value = aws_instance.public.id
}

output "public_instance_ip" {
  value = aws_instance.public.public_ip
}

output "private_instance_id" {
  value = aws_instance.private.id
}

output "private_instance_private_ip" {
  value = aws_instance.private.private_ip
}

```

## 6. Root `outputs.tf`

```

output "vpc_id" {
  value = module.vpc.vpc_id
}

output "nat_gateway_id" {
  value = module.vpc.nat_gateway_id
}

output "public_subnet_id" {
  value = module.vpc.public_subnet_1a_id
}

output "private_subnet_id" {
  value = module.vpc.private_subnet_1a_id
}

output "public_instance_id" {
  value = module.ec2.public_instance_id
}

output "public_instance_ip" {
  value = module.ec2.public_instance_ip
}

output "private_instance_id" {
  value = module.ec2.private_instance_id
}

output "private_instance_private_ip" {
  value = module.ec2.private_instance_private_ip
}

```

## 7. Deploy the infrastructure

Open CMD or PowerShell in your project folder.

```

terraform init
terraform fmt -recursive
terraform validate
terraform plan
terraform apply

```

Type `yes` when prompted. Terraform will create the resources and display the instance IDs and IP addresses.

To remove the infrastructure when finished practicing:

```
terraform destroy
```

Important cost note: A NAT Gateway has ongoing hourly and data-processing charges. This configuration creates one NAT Gateway shared by both private subnets. For a practice environment, run `terraform destroy` when you no longer need the resources.&#x20;
