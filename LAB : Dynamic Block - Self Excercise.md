# Creating an AWS Security Group with Terraform and Configuring Dynamic Ingress and Egress Rules
Dynamic blocks in Terraform allow you to define repeated nested blocks dynamically within a resource or module configuration based on dynamic input.
```
cd ~
mkdir dynamic_block_lab && cd dynamic_block_lab
```
Now Create Files ie. `provider.tf,` `vars.tf,` `terraform.tfvars,`  `sg.tf.`
```
vi provider.tf
```
Add the given lines, by pressing "INSERT" 
```
provider "aws" {
  region = "us-east-1"
}   
```
Save the file using "ESCAPE + :wq!"
```
vi vars.tf
```
```
variable "ingress_ports" {
  type = list(number)
}

variable "egress_ports" {
  type = list(number)
}

```
Save the file using "ESCAPE + :wq!"
 
```
vi sg.tf
```
Add the given lines, by pressing "INSERT" 
```
resource "aws_security_group" "allow_ports" {
  name = "dev_sg"

  dynamic "ingress" {
    for_each = var.ingress_ports
    iterator = port
    content {
      from_port   = port.value
      to_port     = port.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  dynamic "egress" {
    for_each = var.egress_ports
    content {
      from_port   = egress.value
      to_port     = egress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  tags = {
    Name = "dev_sg"
  }
}
```
Save the file using "ESCAPE + :wq!"

```
vi terraform.tfvars
```
Add the given lines, by pressing "INSERT" 
```
# Define a list of ingress ports
ingress_ports = [443, 22, 8080, 8082, 9090]

# Define a list of egress ports
egress_ports = [7000, 8080, 8082, 9292]
```
Save the file using "ESCAPE + :wq!"

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
terraform apply -auto-approve
```
Once applied, go to the Console and check that the new **Security Group** is created.

Use the `terraform destroy` command to clean the resources created in this lab
```
terraform destroy -auto-approve
```
Once Done remove the Directory as well.
```
cd ~
rm -rf dynamic_block_lab
```
