# Terraform Import Commands for AWS Services

| Service | Create with Terraform | Remove State Entry | Import Command |
|---|---|---|---|
| EC2 | `terraform apply` | `terraform state rm aws_instance.web` | `terraform import aws_instance.web i-INSTANCE_ID` |
| S3 | `terraform apply` | `terraform state rm aws_s3_bucket.demo` | `terraform import aws_s3_bucket.demo BUCKET_NAME` |
| VPC | `terraform apply` | `terraform state rm aws_vpc.main` | `terraform import aws_vpc.main vpc-VPC_ID` |
| Subnet | `terraform apply` | `terraform state rm aws_subnet.public` | `terraform import aws_subnet.public subnet-SUBNET_ID` |
| Security Group | `terraform apply` | `terraform state rm aws_security_group.web` | `terraform import aws_security_group.web sg-SG_ID` |
| EBS Volume | `terraform apply` | `terraform state rm aws_ebs_volume.data` | `terraform import aws_ebs_volume.data vol-VOLUME_ID` |
| IAM User | `terraform apply` | `terraform state rm aws_iam_user.dev` | `terraform import aws_iam_user.dev USERNAME` |
| DynamoDB | `terraform apply` | `terraform state rm aws_dynamodb_table.demo` | `terraform import aws_dynamodb_table.demo TABLE_NAME` |
| Lambda | `terraform apply` | `terraform state rm aws_lambda_function.app` | `terraform import aws_lambda_function.app FUNCTION_NAME` |

## Important Notes

1. `terraform apply` creates resources according to your Terraform configuration.
2. `terraform state rm` removes a resource from Terraform state without deleting the actual AWS resource.
3. `terraform import` imports an existing AWS resource into Terraform state.
4. Replace placeholders such as `i-INSTANCE_ID`, `BUCKET_NAME`, and `VPC_ID` with actual resource identifiers.
5. Ensure the corresponding Terraform resource block exists before importing.
6. Run `terraform plan` after importing to verify that your configuration matches the existing resource.
7. Back up your Terraform state before using `terraform state rm`.

**Recommended workflow:**

```bash
terraform init
terraform apply
terraform state list
terraform state rm aws_instance.web
terraform import aws_instance.web i-INSTANCE_ID
terraform state show aws_instance.web
terraform plan
```

**Warning:** This workflow is for practice. Removing a resource from state and re-importing it is usually unnecessary in normal projects. Never apply a plan without checking for unintended resource changes or replacements.

Understood. You want Terraform scripts in exactly this format, using the `import { to = ... id = ... }` block inside the Terraform configuration for your existing AWS resources.

Below are copy-ready examples. Replace the example IDs and names with your actual AWS resource details.

## 1. EC2 instance

`main.tf`

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "myinstance" {
  ami           = "ami-REPLACE_WITH_ACTUAL_AMI_ID"
  instance_type = "t3.micro"
}

import {
  to = aws_instance.myinstance
  id = "i-REPLACE_WITH_ACTUAL_INSTANCE_ID"
}
```

## 2. S3 bucket

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_s3_bucket" "mybucket" {
  bucket = "my-existing-bucket-name"
}

import {
  to = aws_s3_bucket.mybucket
  id = "my-existing-bucket-name"
}
```

## 3. VPC

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_vpc" "myvpc" {
  cidr_block = "10.0.0.0/16"
}

import {
  to = aws_vpc.myvpc
  id = "vpc-REPLACE_WITH_ACTUAL_VPC_ID"
}
```

## 4. Subnet

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_subnet" "mysubnet" {
  vpc_id     = "vpc-REPLACE_WITH_ACTUAL_VPC_ID"
  cidr_block = "10.0.1.0/24"
}

import {
  to = aws_subnet.mysubnet
  id = "subnet-REPLACE_WITH_ACTUAL_SUBNET_ID"
}
```

## 5. Security Group

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_security_group" "mysg" {
  name        = "my-existing-security-group"
  description = "Imported security group"
  vpc_id      = "vpc-REPLACE_WITH_ACTUAL_VPC_ID"
}

import {
  to = aws_security_group.mysg
  id = "sg-REPLACE_WITH_ACTUAL_SECURITY_GROUP_ID"
}
```

## 6. EBS volume

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_ebs_volume" "myvolume" {
  availability_zone = "ap-south-1a"
  size              = 10
  type              = "gp3"
}

import {
  to = aws_ebs_volume.myvolume
  id = "vol-REPLACE_WITH_ACTUAL_VOLUME_ID"
}
```

## 7. IAM user

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_iam_user" "myuser" {
  name = "my-existing-iam-user"
}

import {
  to = aws_iam_user.myuser
  id = "my-existing-iam-user"
}
```

## 8. Lambda function

```
provider "aws" {
  region = "ap-south-1"
}

resource "aws_lambda_function" "myfunction" {
  function_name = "my-existing-function"
  role          = "arn:aws:iam::123456789012:role/my-lambda-role"
  handler       = "index.handler"
  runtime       = "python3.12"
  filename      = "lambda.zip"
}

import {
  to = aws_lambda_function.myfunction
  id = "my-existing-function"
}
```

For Lambda, the configuration must match the existing function; the example's role, runtime, handler, and package settings are placeholders.

## 9. Commands to execute

Save the configuration as `main.tf`, then run:

```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

The `import` block tells Terraform which existing resource to import. Review the plan before applying; Terraform can also propose changes if your configuration doesn't match the actual resource.

Note: The `import {}` block approach requires Terraform 1.5 or later. You can use multiple import blocks in one configuration file, provided each targets a distinct resource and each resource has a matching configuration.
