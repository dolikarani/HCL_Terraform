## AWS EC2 instance creation using Terraform Input Variables


### Task-1: Create EC2 instance using variables 
```
mkdir variables-lab && cd variables-lab/
```
Now Create four Files ie. `provider.tf,` `vars.tf,` `terraform.tfvars,` `instance.tf.`
```
vi provider.tf
```
Add the given lines, by pressing "INSERT" 
```
provider "aws"{
  access_key=var.AWS_ACCESS_KEY
  secret_key=var.AWS_SECRET_KEY
  region=var.AWS_REGION
}
```
Save the file using "ESCAPE + :wq!"
```
vi vars.tf
```
Add the given lines, by pressing "INSERT" also replace your `Region`
```
variable "AWS_ACCESS_KEY"{}
variable "AWS_SECRET_KEY"{}
variable "AWS_REGION"{
  default = "us-east-2"
}
```
Save the file using "ESCAPE + :wq!"
```
vi terraform.tfvars
```
Add the given lines, by pressing "INSERT" Also replace them with your `AccessKey` and `Secret Access Keys` that you created and downloaded earlier.
```
AWS_ACCESS_KEY="< Insert your AWS Access Key >"
AWS_SECRET_KEY="< Insert your AWS Secret Key >"
```
Save the file using "ESCAPE + :wq!"
```
vi instance.tf
```
Add the given lines, by pressing "INSERT" Also replace your `region's AMI ID` and `YourName`
```
resource "aws_instance" "terraform_example"{
  ami = "ami-07efac79022b86107"
  instance_type="t2.micro"
  tags = {
    Name = "Variables-Lab2-YourName"
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
Once applied, go to the Console and check that the new EC2 is created using Variables.

Use the `"terraform destroy"` command to clean the infrastructure used in this lab
```
terraform destroy -auto-approve
```

### Task-2: Implementing map variables that dynamically fetch AMI based on the Linux distro selected
```
vi instance.tf
```
Delete all the existing lines and Add the given code, by pressing "INSERT" and replacing `YourName`

`Note:` To delete all the lines at a time use the commands `Esc+gg+dG` and once cleared then add the new lines.
```
resource "aws_instance" "terraform_example"{
  ami = var.AMIS[var.Linux_distro]
  instance_type="t2.micro"
  tags = {
    Name = "YourName-Lab2-Task2"
  }
}
```
Save the file using "ESCAPE + :wq!"

Now, Also make changes in the `Vars.tf` file.
```
vi vars.tf
```
Delete all existing lines and Add the given lines, by pressing "INSERT" Also ensure to replace your `Region,` `AMi IDs` of the same region for `redhat,` `ubuntu` and `amazon`

`Note:` To delete all the lines at a time use `Esc+gg+dG`
```
variable "AWS_ACCESS_KEY"{}
variable "AWS_SECRET_KEY"{}
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
   redhat="ami-0ba62214afa52bec7"
   ubuntu="ami-0fb653ca2d3203ac1"
   amazon="ami-0231217be14a6f3ba"
  }
}
```
Save the file using "ESCAPE + :wq!"
```
terraform fmt
```
```
terraform validate
```
```
terraform plan -var 'Linux_distro=redhat' -out myplan
```
```
terraform apply myplan
```
Use the `terraform destroy` command to clean the infrastructure used in this lab
```
terraform destroy
```
Once Done remove the `EC2-lab` Directory.
```
cd ~
rm -rf variables-lab

```

