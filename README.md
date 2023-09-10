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


## Let's Start With Real Life Examples!

1. Create Working directory, I created TF-UCS

2. create vars.tf
```
variable "aws_region" {
  default     = "us-east-2"
}
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
  default     = 2
}
variable "ami_key_pair_name" {
  default = "TF-UCS"
}
variable "security_group" {
  default = "web-server-sg-tf"
  }
```

3. Create provider.tf
```
provider "aws" {
  region     = var.aws_region
}
```

4. Now we have two files, we are ready to init
```
$ terraform init
```

5. After initilization .terraform file get created
   
6. Create VPC (vpc.tf)
```
resource "aws_vpc" "prod_vpc" {
    cidr_blocks         = "10.0.0.0/16"
    enable_dns_support  = "true"
    enable_dns_hostname = "true"
    enable_classiclink  = "true"
    instance_tenancy = “default”
    tags = {
     Name = "prod_vpc"
   }
  }
```
7. Create public subnet (vpc.tf)
```
resource “aws_subnet” “prod-subnet-public-1” {
    vpc_id = “${aws_vpc.prod-vpc.id}”
    cidr_block = “10.0.1.0/24”
    map_public_ip_on_launch = “true” //it makes this a public subnet
    availability_zone = “eu-west-2a”
    tags {
        Name = “prod-subnet-public-1”
    }
}
```
9. Create Internat Gateway (network.tf)
```
resource "aws_internet_gateway" "prod-igw" {
    vpc_id = "${aws_vpc.prod-vpc.id}"
    tags {
        Name = "prod-igw"
    }
}
```
11. Create Custom Route Table for public subnet (network.tf)
```
resource "aws_route_table" "prod-public-crt" {
    vpc_id = "${aws_vpc.main-vpc.id}"
    
    route {
        //associated subnet can reach everywhere
        cidr_block = "0.0.0.0/0" 
        //CRT uses this IGW to reach internet
        gateway_id = "${aws_internet_gateway.prod-igw.id}" 
    }
    
    tags {
        Name = "prod-public-crt"
    }
}
```
13. Associate CRT and Subnet (network.tf)
```
resource "aws_route_table_association" "prod-crta-public-subnet-1"{
    subnet_id = "${aws_subnet.prod-subnet-public-1.id}"
    route_table_id = "${aws_route_table.prod-public-crt.id}"
}
```
15. Create Security Group (network.tf)
```
resource "aws_security_group" "ssh-allowed" {
    vpc_id = "${aws_vpc.prod-vpc.id}"
    
    egress {
        from_port = 0
        to_port = 0
        protocol = -1
        cidr_blocks = ["0.0.0.0/0"]
    }
    ingress {
        from_port = 22
        to_port = 22
        protocol = "tcp"
        // This means, all ip address are allowed to ssh ! 
        // Do not do it in the production. 
        // Put your office or home address in it!
        cidr_blocks = ["0.0.0.0/0"]
    }
    //If you do not add this rule, you can not reach the NGIX  
    ingress {
        from_port = 80
        to_port = 80
        protocol = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }
    tags {
        Name = "ssh-allowed"
    }
}
```
17. Add AMI varible into vars.tf
```
variable "AMI" {
    type = "map"
    
    default {
        eu-west-2 = "ami-03dea29b0216a1e03"
        us-east-1 = "ami-0c2a1acae6667e438"
    }
}
```
19. Create EC2
```
resource "aws_instance" "web1" {
    ami = "${lookup(var.AMI, var.AWS_REGION)}"
    instance_type = "t2.micro"
    # VPC
    subnet_id = "${aws_subnet.prod-subnet-public-1.id}"
    # Security Group
    vpc_security_group_ids = ["${aws_security_group.ssh-allowed.id}"]
    # the Public SSH key
    key_name = "${aws_key_pair.london-region-key-pair.id}"
    # nginx installation
    provisioner "file" {
        source = "nginx.sh"
        destination = "/tmp/nginx.sh"
    }
    provisioner "remote-exec" {
        inline = [
             "chmod +x /tmp/nginx.sh",
             "sudo /tmp/nginx.sh"
        ]
    }
    connection {
        user = "${var.EC2_USER}"
        private_key = "${file("${var.PRIVATE_KEY_PATH}")}"
    }
}
// Sends your public key to the instance
resource "aws_key_pair" "london-region-key-pair" {
    key_name = "london-region-key-pair"
    public_key = "${file(var.PUBLIC_KEY_PATH)}"
}
```
21. Create key-pair
```
$ ssh-keygen -f tf-demo-key=pair
```
23. We are ready - Create resources
```
$ terraform plan -out terraform.out
$ terraform apply terraform.out
```
24. Verification

Browse the public IP of the EC2 instance, it should show NGNIX welcome page - Welcome to ngnix!

26. Destroy
```
$ terraform apply -destroy
```
