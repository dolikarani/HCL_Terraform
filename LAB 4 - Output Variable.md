## Using Output Feature 

### Task-1: Using output feature of Terraform to get the IP Address of EC2 Instance
```
mkdir output-variable-lab && cd output-variable-lab
```
```
ls
```
Create the instance.tf file
```
vi instance.tf
```
```
provider "aws" {
  region     = var.AWS_REGION
}

resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = var.Instance_type
  provisioner "local-exec" {
    command = "echo ${aws_instance.example.private_ip} >> private_ips.txt"
  }
}
```
Save the changes
```
vi output.tf
```
```
output "Public_ip" {
  description = "Public IP of the instance"
  value = aws_instance.example.public_ip
}

output "Private_ip" {
  sensitive = true
  description = "Private IP of the instance"
  value = aws_instance.example.private_ip

}
```
Save the file

In `vars.tf` ensure to replace your `region` and Include your `region's Ubuntu AMI ID` to the list.
```
vi vars.tf
```
```
variable "Instance_type"{
  description = "Instance type for the EC2 Instance"
  default = "t2.micro"
}

variable "AWS_REGION"{
  default = "us-east-2"
}

variable "Linux_distro"{
  #description = "Please Enter the Linux distro (redhat, ubuntu, amazon)"
  default = "amazon"
  }

variable "AMIS"{
  type=map(string)
  default={
   redhat="ami-0d77c9d87c7e619f9"
   ubuntu="ami-09040d770ffe2224f"
   amazon="ami-0ddda618e961f2270"
  }
}
```
Once replaced, save the changes.
```
terraform init
```
```
terraform fmt
```
```
terraform validate
```
```
terraform plan
```
```
terraform apply
```
```
ls 
cat private_ips.txt 
```
```
terraform output Public_ip
```
```
terraform output Private_ip
```
Use the `terraform destroy` command to clean the infrastructure used in this lab.
```
terraform destroy
```
Once done, remove the directory and Zip file using the `rm -rf` command below.
```
cd ~
rm -rf output-variable-lab
```


