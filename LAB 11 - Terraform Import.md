## Understanding Terraform Import
Before performing the Terraform import lab, you'll need to create an EC2 instance manually in the AWS console to obtain the necessary information like the AMI ID, instance type, and any tags you want to associate with the instance.
Once the instance is running, you can run the following step to understand Terraform.
```
mkdir import_lab && cd import_lab
```
```
vi import.tf
```
Add the given lines, by pressing "INSERT" 
```
provider "aws" {
  region = "us-east-1"
}   

resource "aws_instance" "test_instance" {
  ami = "<***AMI_ID OF THE EC2 Instance to be IMPORTED***>"
  instance_type = "<***INSTANCE TYPE***>"
  tags = {
    name = "<TAGS if ANY / New Tags can be added as well>"
  }
}
```
Save the file using "ESCAPE + :wq!"
```
terraform init
```
Replace the **Resource-ID** in the below command and execute
```
terraform import aws_instance.test_instance <***Resource-ID***>
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
cd ..
```
```
rm -rf import_lab
```

