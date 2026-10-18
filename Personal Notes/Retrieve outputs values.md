In Terraform, you use an `output` block to display resource details after `terraform apply`. For example, EC2 can display its public and private IP addresses, while S3 displays the bucket name and ARN, and VPC displays the VPC ID and CIDR block.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://developer.hashicorp.com\&sz=32)

HashiCorp Developer

+1



Here are copy-ready Terraform scripts for EC2, S3, VPC, Subnet, Security Group, EBS, IAM, and RDS, including the commands to apply the configuration and retrieve each output.

# 1. EC2 — Public IP, Private IP, Instance ID

Create `main.tf`:

```
provider "aws" {
  region = "ap-south-1"
}

data "aws_ami" "linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.linux.id
  instance_type = "t3.micro"

  tags = {
    Name = "Terraform-EC2"
  }
}

output "ec2_instance_id" {
  value = aws_instance.web.id
}

output "ec2_public_ip" {
  value = aws_instance.web.public_ip
}

output "ec2_private_ip" {
  value = aws_instance.web.private_ip
}

output "ec2_availability_zone" {
  value = aws_instance.web.availability_zone
}
```

Run:

```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

Retrieve individual values:

```
terraform output ec2_instance_id
terraform output ec2_public_ip
terraform output ec2_private_ip
terraform output ec2_availability_zone
```

Retrieve all outputs:

```
terraform output
```

Note: A public IP is assigned only when the instance's network configuration supports public IP assignment. A private IP is assigned within its subnet.

# 2. S3 — Bucket Name, ARN, Regional Domain

Create `main.tf` in a separate folder:

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_s3_bucket" "demo" {
  bucket = "replace-with-your-globally-unique-bucket-name"

  tags = {
    Name = "Terraform-S3"
  }
}

output "s3_bucket_name" {
  value = aws_s3_bucket.demo.bucket
}

output "s3_bucket_arn" {
  value = aws_s3_bucket.demo.arn
}

output "s3_bucket_region" {
  value = aws_s3_bucket.demo.region
}

output "s3_bucket_domain" {
  value = aws_s3_bucket.demo.bucket_regional_domain_name
}
```

Run commands:

```
terraform init
terraform validate
terraform plan
terraform apply
```

Retrieve outputs:

```
terraform output s3_bucket_name
terraform output s3_bucket_arn
terraform output s3_bucket_region
terraform output s3_bucket_domain
terraform output
```

Replace the bucket name before applying; S3 bucket names must be globally unique.

# 3. VPC — VPC ID, CIDR, ARN

Create `main.tf` in a separate folder:

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "Terraform-VPC"
  }
}

output "vpc_id" {
  value = aws_vpc.main.id
}

output "vpc_cidr_block" {
  value = aws_vpc.main.cidr_block
}

output "vpc_arn" {
  value = aws_vpc.main.arn
}

output "vpc_default_route_table_id" {
  value = aws_vpc.main.default_route_table_id
}
```

Run commands:

```
terraform init
terraform validate
terraform plan
terraform apply
```

Retrieve outputs:

```
terraform output vpc_id
terraform output vpc_cidr_block
terraform output vpc_arn
terraform output vpc_default_route_table_id
terraform output
```

# 4. Subnet — Subnet ID, CIDR, Availability Zone

Create `main.tf` in a separate folder:

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "Terraform-VPC"
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "ap-south-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "Terraform-Public-Subnet"
  }
}

output "subnet_id" {
  value = aws_subnet.public.id
}

output "subnet_cidr_block" {
  value = aws_subnet.public.cidr_block
}

output "subnet_availability_zone" {
  value = aws_subnet.public.availability_zone
}

output "subnet_vpc_id" {
  value = aws_subnet.public.vpc_id
}
```

Run commands:

```
terraform init
terraform validate
terraform plan
terraform apply
```

Retrieve outputs:

```
terraform output subnet_id
terraform output subnet_cidr_block
terraform output subnet_availability_zone
terraform output subnet_vpc_id
terraform output
```

`map_public_ip_on_launch = true` enables public IP assignment for supported instances launched in the subnet; an Internet Gateway and suitable route are also needed for internet connectivity.

# 5. Security Group — ID, ARN, VPC ID

Create `main.tf` in a separate folder:

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "Terraform-VPC"
  }
}

resource "aws_security_group" "web" {
  name        = "terraform-web-sg"
  description = "Allow HTTP traffic"
  vpc_id      = aws_vpc.main.id

  ingress {
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
    Name = "Terraform-Web-SG"
  }
}

output "security_group_id" {
  value = aws_security_group.web.id
}

output "security_group_arn" {
  value = aws_security_group.web.arn
}

output "security_group_vpc_id" {
  value = aws_security_group.web.vpc_id
}
```

Run commands:

```
terraform init
terraform validate
terraform plan
terraform apply
```

Retrieve outputs:

```
terraform output security_group_id
terraform output security_group_arn
terraform output security_group_vpc_id
terraform output
```

Security note: This example allows HTTP from any IPv4 address. For production, restrict ingress to trusted IP ranges where possible.

# 6. EBS Volume — Volume ID, Size, Availability Zone

Create `main.tf` in a separate folder:

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_ebs_volume" "data" {
  availability_zone = "ap-south-1a"
  size              = 10
  type              = "gp3"
  encrypted         = true

  tags = {
    Name = "Terraform-EBS"
  }
}

output "ebs_volume_id" {
  value = aws_ebs_volume.data.id
}

output "ebs_volume_size" {
  value = aws_ebs_volume.data.size
}

output "ebs_volume_az" {
  value = aws_ebs_volume.data.availability_zone
}

output "ebs_volume_arn" {
  value = aws_ebs_volume.data.arn
}
```

Run commands:

```
terraform init
terraform validate
terraform plan
terraform apply
```

Retrieve outputs:

```
terraform output ebs_volume_id
terraform output ebs_volume_size
terraform output ebs_volume_az
terraform output ebs_volume_arn
terraform output
```

This creates a standalone EBS volume; it does not automatically attach it to an EC2 instance. The volume and the instance must be in the same Availability Zone to attach them.

# 7. IAM User — Username, ARN, Unique ID

Create `main.tf` in a separate folder:

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_iam_user" "dev" {
  name = "terraform-demo-user"

  tags = {
    Name = "Terraform-IAM-User"
  }
}

output "iam_user_name" {
  value = aws_iam_user.dev.name
}

output "iam_user_arn" {
  value = aws_iam_user.dev.arn
}

output "iam_user_unique_id" {
  value = aws_iam_user.dev.unique_id
}
```

Run commands:

```
terraform init
terraform validate
terraform plan
terraform apply
```

Retrieve outputs:

```
terraform output iam_user_name
terraform output iam_user_arn
terraform output iam_user_unique_id
terraform output
```

This creates an IAM user without access keys or permissions. Grant only the permissions the user actually needs.

# 8. RDS — Database Identifier, Endpoint, Port

This example creates a small MySQL RDS instance for practice. RDS can incur charges, so use it only when needed.

Create `main.tf` in a separate folder:

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_db_instance" "mysql" {
  identifier             = "terraform-demo-mysql"
  engine                 = "mysql"
  engine_version         = "8.0"
  instance_class         = "db.t3.micro"
  allocated_storage      = 20
  username               = "dbadmin"
  password               = "ChangeThisPassword123!"
  db_name                = "appdb"
  skip_final_snapshot    = true
  publicly_accessible    = false
  backup_retention_period = 0

  tags = {
    Name = "Terraform-RDS"
  }
}

output "rds_identifier" {
  value = aws_db_instance.mysql.identifier
}

output "rds_endpoint" {
  value = aws_db_instance.mysql.endpoint
}

output "rds_address" {
  value = aws_db_instance.mysql.address
}

output "rds_port" {
  value = aws_db_instance.mysql.port
}

output "rds_arn" {
  value = aws_db_instance.mysql.arn
}
```

Run commands:

```
terraform init
terraform validate
terraform plan
terraform apply
```

Retrieve outputs:

```
terraform output rds_identifier
terraform output rds_endpoint
terraform output rds_address
terraform output rds_port
terraform output rds_arn
terraform output
```

Important: This minimal RDS example may need additional configuration, such as a DB subnet group and VPC security group, depending on your AWS environment and provider requirements. For real projects, use a Secrets Manager-managed password instead of a hardcoded password.

## 9. Common output commands — quick reference

Terraform outputs can be queried individually or together from the command line.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://developer.hashicorp.com\&sz=32)

HashiCorp Developer

+1



| Requirement            | Command                                 |
| ---------------------- | --------------------------------------- |
| Show all outputs       | `terraform output`                      |
| Show EC2 public IP     | `terraform output ec2_public_ip`        |
| Show EC2 private IP    | `terraform output ec2_private_ip`       |
| Show S3 bucket name    | `terraform output s3_bucket_name`       |
| Show VPC ID            | `terraform output vpc_id`               |
| Show subnet ID         | `terraform output subnet_id`            |
| Show security group ID | `terraform output security_group_id`    |
| Show EBS volume ID     | `terraform output ebs_volume_id`        |
| Show IAM user ARN      | `terraform output iam_user_arn`         |
| Show RDS endpoint      | `terraform output rds_endpoint`         |
| Show raw string output | `terraform output -raw ec2_public_ip`   |
| Save outputs to JSON   | `terraform output -json > outputs.json` |

Remember: Keep each standalone example in a separate folder, and use `terraform destroy` in that folder when you finish practicing. Review the plan before applying or destroying resources.
