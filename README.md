# Terrraform_Tutorials

### Pre-requisite:

**1) Terraform server (EC2-Instance):**
```
# Install yum-config-manager to manage your repositories.
sudo yum install -y yum-utils
# Use yum-config-manager to add the official HashiCorp Linux repository
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
# Install Terraform
sudo yum -y install terraform
```
or 

Install Terraform in your Windows PC / Laptop


**2) Connect to AWS**
```
export AWS_ACCESS_KEY_ID=xxxxx
export AWS_SECRET_ACCESS_KEY=xxxxx
export AWS_DEFAULT_REGION=us-west-2

# Verify Environment Variables
echo $USER   #system variable
echo $AWS_ACCESS_KEY
```
### Examples:

**1) EC2 Instance**

myfirsttfvm.tf

```
provider "aws" {
  region     = "us-east-2"
  access_key = "xxxxxx"
  secret_key = "xxxxxx"
}
resource "aws_instance" "myfirsttfvm" {
  ami             = "ami-05842f1afbf311a43"
  instance_type   = "t2.micro"
  key_name        = "Terraform01"
  security_groups = ["my-tf-sg"]
  tags            = {
    Name = "HelloWorld"
  }
}
```

$ terraform init
$ terraform plan
$ terraform fmt
$ terraform validate
$ terraform apply -auto-approve

**2) EC2 Instance using Variables**

vars.tf
```
variable "ami"{
 description="amazon machine image value"
 default="ami-05842f1afbf311a43"
}
variable "instance_type"{
 description="amazon instance type"
 default="t2.micro"
}
variable "instance_count"{
 description="amazon instance count"
 default="2"
}
```

myfirsttfvm.tf
```
provider "aws" {
  region     = "us-east-2"
  access_key = "xxxxxxx"
  secret_key = "xxxxxxx"
}
resource "aws_instance" "myfirsttfvm" {
  count           = "$var.instance_count"
  ami             = "$var.ami"
  instance_type   = "$var.instance_type"
  key_name        = "Terraform01"
  security_groups = ["my-tf-sg"]
  tags = {
   Name = "mytfvm - ${count.index}"
   }
}
```

$ terraform plan
$ terraform apply -auto-approve  >> 1 added 1 changed

**3) EC2 Instance using Declarative Variables**

Can remove instance_count variable fron vars.tf file and pass like below command

$ terraform apply -var instance_count=3 -auto-approve

```
# this is single line comment in terraform
/* this is 
   multiple line 
   comment in terraform */
```

### 4) EC2 Instance without Hardcoring Access Keys**

vars.tf
```
variable "ami"{
 description="amazon machine image value"
 default="ami-05842f1afbf311a43"
}
variable "instance_type"{
 description="amazon instance type"
 default="t2.micro"
}
variable "instance_count"{
 description="amazon instance count"
 default="3"
}
```

myfirsttfvm.tf
```
provider "aws" {
  region     = "us-east-2"
}
resource "aws_instance" "AWSServer" {
  ami             = "${var.ami}"
  instance_type   = "${var.instance_type}"
  count           = "${var.instance_count}"
  key_name        = "Terraform01"
  security_groups = ["my-tf-sg"]
  tags = {
    Name = "myTFvm - ${count.index}"
   }
}
```

$ sudo su
$ terraform plan
$ terraform apply -var instance_count=4 -auto-approve


### 5) EC2 Instance using User Data

installHttpd.sh
```
#! /bin/bash -ex
yum update -y
yum install -y httpd
echo "<h2>Welcome to devops Training !!! </h2>" >> /var/www/html/index.html
service httpd start
chkconfig httpd on
```

$ chmod u+x installHttpd.sh

myfirsttfvm.tf
```
provider "aws" {
  region     = "us-east-2"
}
resource "aws_instance" "web" {
  ami             = "${var.ami}"
  instance_type   = "${var.instance_type}"
  count           = "${var.instance_count}"
  key_name        = "Terraform01"
  security_groups = ["my-tf-sg"]
  user_data       = "${file("installHttpd.sh")}"
  tags = {
    Name = "myTFwebVM - ${count.index}"
  }
}
```

$ sudo su
$ terraform plan
$ terraform apply -auto-approve

### 6) Multiple EC2 Instances using User Data and Instance name
Pre-requisite: Create new folder named EC2Instances and update profile using command **aws configure**

main.tf
```
provider "aws" {
  profile = "default"
  region  = "us-east-2"
}
```

vars.tf
```
variable "instance_name" {
  description = "Name of the instance to be created"
  default     = "Terraform"
}
variable "instance_type" {
  default = "t2.micro"
}
variable "ami_id" {
  description = "The AMI to use"
  default     = "ami-05842f1afbf311a43"
}
variable "number_of_instances" {
  description = "number of instances to be created"
  default     = 3
}
variable "ami_key_pair_name" {
  default = "cts-demo"
}
variable "security_group" {
  default = "web-server-sg-tf"
  }
```

sg.tf
```
resource "aws_security_group" "web_server_sg_tf" {
  name        = "web-server-sg-tf"
  description = "Allow HTTP to web server"
  vpc_id      = "vpc-07b2af77184cf668e"
  ingress {
    description = "HTTPS ingress"
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
}
```

installHttpd.sh
```
#! /bin/bash -ex
yum update -y
yum install -y httpd
echo "<h2> Welcome to Terraform !!! </h2>" >> /var/www/html/index.html
service httpd start
chkconfig httpd on
```

ec2.tf
```
resource "aws_instance" "TFDemo3" {
  ami             = var.ami_id
  instance_type   = var.instance_type
  count           = var.number_of_instances
  key_name        = var.ami_key_pair_name
  user_data       = file("installHttpd.sh")
  security_groups = [var.security_group]
  tags = {
    Name = "${var.instance_name}_${count.index}"
  }
}
```

$ sudo su
$ terraform plan
$ terraform apply -auto-approve >> Apply complete! Resources: 4 added, 0 changed, 0 destroyed

### 7) S3 bucket

Pre-requisite: verify S3 bucket permission (IAM policy)

myfirsttfbucket.tf
```
provider "aws" {
  region = "us-east-2"
}
resource "aws_s3_bucket" "mytfbucket00001" {
  bucket = "mytfbucket00001"
  acl    = "private"
  versioning {
    enabled = "true"
  }
}
```
$ sudo su
$ terraform plan
$ terraform apply -auto-approve

### 8) Mysql DB
-------------
verify RDS permission (IAM policy)

myfirsttfrds.tf
```
provider "aws" {
  region     = "us-east-2"
}
resource "aws_db_instance" "mysqldb01" {
  allocated_storage    = 20
  db_name              = "mysqldb01"
  engine               = "mysql"
  engine_version       = "5.7"
  instance_class       = "db.t3.micro"
  username             = "ajit"
  password             = "Admin123"
  parameter_group_name = "default.mysql5.7"
  skip_final_snapshot  = true
}
```
$ sudo su
$ terraform plan
$ terraform apply -auto-approve

copy script from official document and update according to your requirement

https://registry.terraform.io/providers/hashicorp/aws/latest/docs

// search "azurerm" for Terraform script to create resources in azure cloud

// Terraform Associate Certification Tutorial list

https://developer.hashicorp.com/terraform/tutorials/certification-associate-tutorials-003

**List state files:** $ terraform state list

**Inspect the current state:** $ terraform show

**Destroy Resources:** $ terraform apply -destroy

**Connect to instance using powershell:** $ ssh ec2-user@<Instance_PublicIP> -i cts-demo.pem
