## Understanding Terraform functions 
### Functions
```
cd ~
```
```
mkdir lab6 && cd lab6
```
```
vi provider.tf
```
Add the given lines, by pressing "INSERT" 
```
provider "aws" {
  region     = var.region
}
```
Save the file using "ESCAPE + :wq!"
```
vi vars.tf
```
Add the given lines, by pressing "INSERT" 
```
variable "region" {
  default = "ap-south-1"
}

variable "tags" {
  type = list
  default = ["ec2-1","ec2-2"]
}

variable "ami" {
  type = map
  default = {
    "us-east-1" = "ami-0e001c9271cf7f3b9"         ******ENTER THE AMI VALUE********
    "us-west-1" = "ami-08012c0a9ee8e21c4"         ******ENTER THE AMI VALUE********
    "ap-south-1" = "ami-0f58b397bc5c1f2e8"        ******ENTER THE AMI VALUE********
  }
}
```
Save the file using "ESCAPE + :wq!"

```
vi functions.tf
```
Add the given lines, by pressing "INSERT" 
```
resource "aws_instance" "application-servers" {
   ami = lookup(var.ami,var.region)
   instance_type = "t2.micro"
   count = 2

   tags = {
     Name = element(var.tags,count.index)
   }
}
```
Save the file using "ESCAPE + :wq!"
```
terraform init
```
```
terraform plan
```
```
terraform apply
```
```
terraform destroy
```
```
cd ~
rm -rf lab6
```
