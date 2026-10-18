Terraform code files don't support colored font directly, so I'll use colored emoji markers and comments.

# 1. EC2 — Without Alias

File: `main.tf`

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

# 🟨 DEFAULT PROVIDER — Mumbai
provider "aws" {
  region = "ap-south-1"
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

resource "aws_instance" "web" {
  # No HIGHLIGHT 3 because we use the default provider
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  tags = {
    Name = "Terraform-EC2-Without-Alias"
  }
}

output "instance_id" {
  value = aws_instance.web.id
}
```

# 2. EC2 — With Alias

File: `main.tf`

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

# 🟨 DEFAULT PROVIDER — Mumbai
provider "aws" {
  region = "ap-south-1"
}

provider "aws" {
  # 🩷 HIGHLIGHT 1 — Alias name
  alias = "west"

  # 🩵 HIGHLIGHT 2 — Oregon region
  region = "us-west-2"
}

data "aws_ami" "amazon_linux" {
  # 💚 HIGHLIGHT 3 — Use alias for AMI lookup
  provider = aws.west

  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

resource "aws_instance" "web" {
  # 💚 HIGHLIGHT 3 — Use alias for EC2
  provider = aws.west

  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t3.micro"

  tags = {
    Name = "Terraform-EC2-With-Alias"
  }
}

output "instance_id" {
  value = aws_instance.web.id
}
```

# 3. S3 — Without Alias

File: `main.tf`

```
provider "aws" {
  # 🟨 DEFAULT PROVIDER — Mumbai
  region = "ap-south-1"
}

resource "aws_s3_bucket" "demo" {
  # No HIGHLIGHT 3 — Uses the default provider

  bucket = "replace-with-your-globally-unique-bucket-name"

  tags = {
    Name = "Terraform-S3-Without-Alias"
  }
}

output "bucket_name" {
  value = aws_s3_bucket.demo.bucket
}
```

# 4. S3 — With Alias

File: `main.tf`

```
provider "aws" {
  # 🟨 DEFAULT PROVIDER — Mumbai
  region = "ap-south-1"
}

provider "aws" {
  # 🩷 HIGHLIGHT 1 — Alias name
  alias = "west"

  # 🩵 HIGHLIGHT 2 — Oregon region
  region = "us-west-2"
}

resource "aws_s3_bucket" "demo" {
  # 💚 HIGHLIGHT 3 — Use alias for S3
  provider = aws.west

  bucket = "replace-with-another-globally-unique-bucket-name"

  tags = {
    Name = "Terraform-S3-With-Alias"
  }
}

output "bucket_name" {
  value = aws_s3_bucket.demo.bucket
}
```

# 5. VPC — Without Alias

File: `main.tf`

```
provider "aws" {
  # 🟨 DEFAULT PROVIDER — Mumbai
  region = "ap-south-1"
}

resource "aws_vpc" "main" {
  # No HIGHLIGHT 3 — Uses the default provider

  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "Terraform-VPC-Without-Alias"
  }
}

output "vpc_id" {
  value = aws_vpc.main.id
}
```

# 6. VPC — With Alias

File: `main.tf`

```
provider "aws" {
  # 🟨 DEFAULT PROVIDER — Mumbai
  region = "ap-south-1"
}

provider "aws" {
  # 🩷 HIGHLIGHT 1 — Alias name
  alias = "west"

  # 🩵 HIGHLIGHT 2 — Oregon region
  region = "us-west-2"
}

resource "aws_vpc" "main" {
  # 💚 HIGHLIGHT 3 — Use alias for VPC
  provider = aws.west

  cidr_block           = "10.1.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "Terraform-VPC-With-Alias"
  }
}

output "vpc_id" {
  value = aws_vpc.main.id
}
```

## Color and highlight reference

| Highlight      | Terraform code          | Meaning                          |
| -------------- | ----------------------- | -------------------------------- |
| 🩷 HIGHLIGHT 1 | `alias = "west"`        | Names the provider configuration |
| 🩵 HIGHLIGHT 2 | `region = "us-west-2"`  | Selects the Oregon region        |
| 💚 HIGHLIGHT 3 | `provider = aws.west`   | Connects a resource to the alias |
| 🟨 Default     | `region = "ap-south-1"` | Default provider region: Mumbai  |

Run the scripts:

```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

Keep each example in a separate folder. Replace the example S3 bucket names before running. Ensure your AWS credentials are configured and that you have permission to create the resources.
