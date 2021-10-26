

### Terraform code

config.tf
```
provider "aws" {
  region = var.aws_region
  ignore_tags {
    key_prefixes = ["managed:"]
  }
}

terraform {
  required_version = ">= 0.13.2"
  # backend "s3" {}
}
```

output.tf
```
output "utility_server_id" {
   value = [aws_instance.mycomp_myproject_utility.*.id]
}

output "Administrator_Password" {
  value = [
    for x in aws_instance.mycomp_myproject_utility.* : rsadecrypt(x.password_data, file(var.myproject_utility_pem)) 
  ]
}
```

util-server.tf
```
# data "template_file" "init" {
#   /*template = "${file("user_data")}"*/
#   template = <<EOF
#  <script>
#    winrm quickconfig -q & winrm set winrm/config/winrs @{MaxMemoryPerShellMB="300"} & winrm set winrm/config @{MaxTimeoutms="1800000"} & winrm set winrm/config/service @{AllowUnencrypted="true"} & winrm set winrm/config/service/auth @{Basic="true"}
#  </script>
#  <powershell>
#    netsh advfirewall firewall add rule name="WinRM in" protocol=TCP dir=in profile=any localport=5985 remoteip=any localip=any action=allow
#    $admin = [ADSI]("WinNT://./administrator, user")
#    $admin.SetPassword("${var.admin_password}")
#    iwr -useb https://omnitruck.chef.io/install.ps1 | iex; install -project chefdk -channel stable -version 0.16.28
#  </powershell>
#  EOF

#   vars = {
#     admin_password = "${var.admin_password}"
#   }
# }

resource "aws_instance" "mycomp_myproject_utility" {
  instance_type = var.aws_instance_type
  ami           = var.aws_ami
  count = var.utility_instance_count
  tags = merge(var.tags,
    {
      Name          = "mycomp-myproject-utility-${count.index + 1}"
      "ssm-os"      = "windows"
      "ssm-enabled" = "true"
  })
  key_name               = var.key_name
  tenancy                = "dedicated"
  subnet_id              = var.aws_subnet_id
  vpc_security_group_ids = [aws_security_group.mycomp_myproject_utility_sg.id]
  iam_instance_profile   = var.utility_iam_instance_profile
  root_block_device {
    encrypted   = true
    kms_key_id  = var.ebs_kms_key
    volume_size = 100
    tags        = var.tags
  }
  get_password_data = "true"

  # user_data = file("./files/bootstrap.txt")
}
```

var.tf
```
variable "environment" {
  description = "test uat prod"
  type        = string
}

variable "aws_region" {
  description = "The AWS region to deploy to (e.g. us-west-2)"
  type        = string
}

variable "tags" {
  description = "S3 Tags"
  type        = map(any)
}

variable "admin_password" {
  description = "Windows Administrator password to login as."
  type        = string
  default     = "myproject#2021"
}

variable "aws_ami" {
  description = "Windows server AMI"
  type        = string
}

variable "key_name" {
  description = "Name of the SSH keypair to use in AWS."
  default     = "myproject-utility"
}

variable "aws_instance_type" {
  description = "myproject Windows utilityity Server Instance Type"
  type        = string
  default     = "c4.large"
}

variable "aws_subnet_id" {
  description = "AWS myproject subnet ID"
  type        = string
  default     = "subnet-xxxx"
}

variable "vpc_id" {
  description = "AWS myproject VPC ID"
  type        = string
  default     = "vpc-xxxx"
}

variable "utility_iam_instance_profile" {
  description = "AWS myproject utility IAM Instance Profile"
  type        = string
  default     = "SSMInstanceProfile"
}

variable "ebs_kms_key" {
  description = "KMS key to use for encryption of EBS volumes"
  type        = string
}

variable "myproject_utility_pem" {
  description = "EC2 pem file location"
  type        = string
}

variable "utility_instance_count" {
  description = "Utility instance count"
  default = 1
}
```

vpc.tf
```

resource "aws_security_group" "mycomp_myproject_util_sg" {
  vpc_id      = var.vpc_id
  name        = "mycomp-myproject-util-sg"
  description = "myproject VPC Security Group"

  # ingress {
  #   from_port   = 3389
  #   to_port     = 3389
  #   protocol    = "tcp"
  #   cidr_blocks = ["0.0.0.0/0"]
  # }  

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

test.tfvar
```
environment = "test"
aws_region  = "ap-southeast-2"

tags = {
  #  "author" = "MyComp-Data-Platform"
  "uuid"  = "xxxx"
  "stack" = "mycomp-myproject-utility"
  # "dataclassification"     = "c3"
  # "dataclassificationdate" = "2020-10-21"
}

aws_subnet_id = "subnet-xxxx"
vpc_id        = "vpc-xxxx"

aws_instance_type = "t3.large"

utility_iam_instance_profile = "SSMInstanceProfile"
ebs_kms_key                  = "arn:aws:kms:ap-southeast-2:333186395126:key/ac766ec0-95e8-4806-80c1-ef5ef24bb1d1" # alias/test-ebs-data
key_name                     = "myproject-utility"

myproject_utility_pem = "/home/york/MyComp/keys/myproject-utility-test.pem"

aws_ami = "ami-0dbcc5e3b0f662f48"

utility_instance_count = 2
```

### Connect via SSM

```
aws ssm start-session --target i-xxxx --region ap-southeast-2 --document-name arn:aws:ssm:ap-southeast-2:xxxx:document/xxxx-SIA-SSMStartRDPSession
```

Then use RDP to connect.



