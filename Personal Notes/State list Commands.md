Here are practical Terraform state command examples using EC2, S3, and VPC. You'll learn how to use `list`, `show`, `rm`, and `mv` with actual resource addresses.

Important: `terraform state rm` and `terraform state mv` change Terraform's state tracking; they do not directly delete or move the real AWS resource.

## 1. Example Terraform script

Create `main.tf`:

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

resource "aws_s3_bucket" "demo" {
  bucket = "replace-with-your-globally-unique-bucket-name"
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
```

Replace the S3 bucket name with a globally unique name. Save this file in a new folder, configure AWS credentials, and run:

```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

After successful creation, your state should contain addresses similar to these:

```
aws_instance.web
aws_s3_bucket.demo
aws_vpc.main
```

## 2. `terraform state list` — list all resources

Command:

```
terraform state list
```

Example output:

```
aws_instance.web
aws_s3_bucket.demo
aws_vpc.main
```

List only EC2 resources:

```
terraform state list aws_instance
```

Save the list to a file on Windows CMD:

```
terraform state list > resources.txt
```

## 3. `terraform state show` — show resource details

Show EC2 details:

```
terraform state show aws_instance.web
```

Show S3 bucket details:

```
terraform state show aws_s3_bucket.demo
```

Show VPC details:

```
terraform state show aws_vpc.main
```

Example output for EC2:

```
# aws_instance.web:
resource "aws_instance" "web" {
    ami           = "ami-xxxxxxxx"
    instance_type = "t3.micro"
    id            = "i-0123456789abcdef0"
}
```

The IDs above are illustrative examples.

## 4. `terraform state rm` — remove a resource from state

Example: stop tracking the EC2 instance in Terraform state.

First, back up your state:

```
terraform state pull > state-backup.json
```

Then run:

```
terraform state rm aws_instance.web
```

Example output:

```
Removed aws_instance.web
Successfully removed 1 resource instance(s).
```

Verify:

```
terraform state list
```

Important: This does not terminate the EC2 instance. The instance remains in AWS. If its resource block is still in `main.tf`, a subsequent `terraform plan` will generally propose creating another instance. Don't apply that plan unless you intend to create it.

## 5. `terraform state mv` — rename a resource in state

Suppose you want to rename the Terraform address from `aws_instance.web` to `aws_instance.app`.

Step 1: Update the resource name in `main.tf`:

```
resource "aws_instance" "app" {
  ami           = data.aws_ami.linux.id
  instance_type = "t3.micro"

  tags = {
    Name = "Terraform-EC2"
  }
}
```

Step 2: Run the state move command:

```
terraform state mv aws_instance.web aws_instance.app
```

Example output:

```
Move "aws_instance.web" to "aws_instance.app"
Successfully moved 1 object(s).
```

Step 3: Verify the new address:

```
terraform state list
```

Expected EC2 entry:

```
aws_instance.app
```

Important: `state mv` changes the resource address in Terraform state, not the actual EC2 instance. Update the configuration references too, such as `aws_instance.web.id` to `aws_instance.app.id`, wherever applicable. Review `terraform plan` afterward to confirm that no unintended replacement is proposed.

## 6. Quick revision table

| Command                        | Purpose                   | Deletes AWS resource? |
| ------------------------------ | ------------------------- | --------------------- |
| `terraform state list`         | Lists tracked resources   | No                    |
| `terraform state show ADDRESS` | Shows resource details    | No                    |
| `terraform state rm ADDRESS`   | Removes state tracking    | No                    |
| `terraform state mv OLD NEW`   | Changes the state address | No                    |
| `terraform state pull`         | Retrieves current state   | No                    |

Practice these commands in a test environment and back up your state before modifying it.
