# 1. Other useful Terraform log files

You can use separate log files for other Terraform configurations, too.
```
set TF_LOG=TRACE
set TF_LOG_PATH=tflogs_terraform_init.logs
terraform init
```


```
set TF_LOG=TRACE
set TF_LOG_PATH=tflogs_terraform_plan.logs
terraform plan
```
```
set TF_LOG=TRACE
set TF_LOG_PATH=tflogs_terraform_destroy.logs
terraform destroy
```

# 2. Linux / Ubuntu / Git Bash commands

If you're using Linux, Ubuntu, or Git Bash, use export instead.
```
export TF_LOG=TRACE
export TF_LOG_PATH="tflogs_ec2.logs"
terraform apply
```
```
export TF_LOG=TRACE
export TF_LOG_PATH="tflogs_ec2_alias.logs"
terraform apply
```
```
export TF_LOG=TRACE
export TF_LOG_PATH="tflogs_vpc.logs"
terraform apply
```
```
export TF_LOG=TRACE
export TF_LOG_PATH="tflogs_vpc_alias.logs"
terraform apply
```
```
export TF_LOG=TRACE
export TF_LOG_PATH="tflogs_s3.logs"
terraform apply
```
```
export TF_LOG=TRACE
export TF_LOG_PATH="tflogs_s3_alias.logs"
terraform apply
```
# 3. To stop Terraform logging in Windows CMD:
```
set TF_LOG=
set TF_LOG_PATH=
```

To stop logging in Linux/Git Bash:
```
unset TF_LOG
unset TF_LOG_PATH
```
# 4. Log levels

Level	Description

ERROR	Errors only

WARN	Warnings and errors

INFO	General information

DEBUG	Detailed debugging

TRACE	Most detailed diagnostic logs
