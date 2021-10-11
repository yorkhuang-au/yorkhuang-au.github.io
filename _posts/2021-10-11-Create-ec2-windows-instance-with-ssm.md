---
layout: post
title: Create ec2 Windows Instance and Connect via SSM
---

### Terraform code

config.tf
```
provider "aws" {
  region = var.aws_region
}

terraform {
  required_version = ">= 0.13.2"
  # backend "s3" {}
}
```

output.tf
```
output "utill_server_id" {
  value = aws_instance.xade_mstr_util.id
}
```

util-server.tf
```
data "template_file" "init" {
  /*template = "${file("user_data")}"*/
  template = <<EOF
 <script>
   winrm quickconfig -q & winrm set winrm/config/winrs @{MaxMemoryPerShellMB="300"} & winrm set winrm/config @{MaxTimeoutms="1800000"} & winrm set winrm/config/service @{AllowUnencrypted="true"} & winrm set winrm/config/service/auth @{Basic="true"}
 </script>
 <powershell>
   netsh advfirewall firewall add rule name="WinRM in" protocol=TCP dir=in profile=any localport=5985 remoteip=any localip=any action=allow
   $admin = [ADSI]("WinNT://./administrator, user")
   $admin.SetPassword("${var.admin_password}")
   iwr -useb https://omnitruck.chef.io/install.ps1 | iex; install -project chefdk -channel stable -version 0.16.28
 </powershell>
 EOF

  vars = {
    admin_password = "${var.admin_password}"
  }
}

resource "aws_instance" "xade_mstr_util" {
  connection {
    type     = "winrm"
    user     = "Administrator"
    password = var.admin_password
  }
  instance_type = var.aws_instance_type
  ami           = var.aws_ami
  tags = merge(var.tags,
    {
      Name          = "xade-mstr-util"
  })
  # key_name             = var.key_name
  tenancy                = "dedicated"
  subnet_id              = var.aws_subnet_id
  vpc_security_group_ids = [aws_security_group.xade_mstr_util_sg.id] 
  iam_instance_profile   = var.util_iam_instance_profile

  user_data = data.template_file.init.rendered
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
  default     = "pasword"
}

variable "aws_ami" {
  default = "ami-0dbcc5e3b0f662f48"
}

variable "key_name" {
  description = "Name of the SSH keypair to use in AWS."
  default     = "AWS Keypair"
}

variable "aws_instance_type" {
  description = "Windows Utility Server Instance Type"
  type        = string
  default     = "c4.large"
}

variable "aws_subnet_id" {
  description = "AWS MSTR subnet ID"
  type        = string
  default     = "subnet-xxxx"
}

variable "vpc_id" {
  description = "AWS MSTR VPC ID"
  type        = string
  default     = "vpc-XXXX"
}

variable "util_iam_instance_profile" {
  description = "AWS MSTR util IAM Instance Profile"
  type        = string
  default     = "SSMInstanceProfile" # with AmazonSSMManagedInstanceCore policy
}
```

vpc.tf
```

resource "aws_security_group" "xade_mstr_util_sg" {
  vpc_id      = var.vpc_id
  name        = "xade-mstr-util-sg"
  description = "Mstr VPC Security Group"

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
  "uuid" = "xxxx"
  # "dataclassification"     = "c3"
  # "dataclassificationdate" = "2020-10-21"
}

aws_subnet_id = "subnet-xxxx"
vpc_id        = "vpc-xxxx"

aws_instance_type = "t3.large"

util_iam_instance_profile = "SSMInstanceProfile"
```

### Connect via SSM

```
aws ssm start-session --target i-xxxx --region ap-southeast-2 --document-name arn:aws:ssm:ap-southeast-2:xxxx:document/xxxx-SIA-SSMStartRDPSession
```

Then use RDP to connect.



