## Launching VPC and EC2 Instance 

### Task-1: Launching VPC and creating subnets
```
cd /home/ubuntu/
```
```
mkdir lab_vpc
```
```
cd lab_vpc/
ls
```
Create vpc.tf and replace your AZs
```
vi vpc.tf
```
```
# For Provider
provider "aws" {
  region = var.AWS_REGION
}

# For VPC
resource "aws_vpc" "main" {
    cidr_block = "10.0.0.0/16"
    instance_tenancy = "default"
    enable_dns_support = "true"
    enable_dns_hostnames = "true"
    tags = {
        Name = "main"
    }
}

# For Subnets
resource "aws_subnet" "main-public-1" {
    vpc_id = aws_vpc.main.id
    cidr_block = "10.0.1.0/24"
    map_public_ip_on_launch = "true"
    availability_zone = "us-east-1a"
    tags = {
        Name = "main-public-1"
    }
}

resource "aws_subnet" "main-private-1" {
    vpc_id = aws_vpc.main.id
    cidr_block = "10.0.4.0/24"
    map_public_ip_on_launch = "true"
    availability_zone = "us-east-1a"
    tags = {
        Name = "main-private-1"
    }
}

# For Internet Gateway
resource "aws_internet_gateway" "main-gw" {
    vpc_id = aws_vpc.main.id
    tags = {
        Name = "main"
    }
} 

# For Route Tables
resource "aws_route_table" "main-public" {
    vpc_id = aws_vpc.main.id
    route {
        cidr_block = "0.0.0.0/0"
        gateway_id = aws_internet_gateway.main-gw.id
    }
    tags = {
        Name = "main-public-1"
    }
}

# For Route associations public
resource "aws_route_table_association" "main-public-1-a" {
    subnet_id = aws_subnet.main-public-1.id
    route_table_id = aws_route_table.main-public.id
}
```
Save the file using "ESCAPE + :wq!"
```
vi nat.tf
```
```
# For NAT Gateway
resource "aws_eip" "nat" {
}

resource "aws_nat_gateway" "nat-gw" {
    allocation_id = aws_eip.nat.id
    subnet_id = aws_subnet.main-public-1.id
    depends_on = [aws_internet_gateway.main-gw]
}

# For Route Table
resource "aws_route_table" "main-private" {
    vpc_id = aws_vpc.main.id
    route {
        cidr_block = "0.0.0.0/0"
        nat_gateway_id = aws_nat_gateway.nat-gw.id
    }
    tags = {
        Name = "main-private-1"
    }
}

# For Route associations private
resource "aws_route_table_association" "main-private-1-a" {
    subnet_id = aws_subnet.main-private-1.id
    route_table_id = aws_route_table.main-private.id
}

```
No Updated need in `nat.tf`. Save the file using "ESCAPE + :wq!"
```
vi vars.tf
```
```
variable "AWS_REGION" {
  default = "us-east-1"
}
```
In `vars.tf` file Update your Region. Save the file using "ESCAPE + :wq!"
```
terraform init
```
```
terraform plan
```
**Note:** In `terraform plan` if it shows any warning just ignore it. But, if it shows any error we need to correct it.

```
terraform apply -auto-approve
```
Once applied, In the Console check that the resources are getting created like `VPC,` `Subnets,` `Internet Gateway` etc....

### Task-2: Launching an EC2 Instance 

Create a `new File` for EC2 Instance
```
vi instance.tf 
```
Add the given lines, by pressing "INSERT" and replace `Yourname-ec2` with your name.
```
resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  # the VPC subnet
  subnet_id = aws_subnet.main-public-1.id
  # the security group
  vpc_security_group_ids = [aws_security_group.allow-ssh.id]
  # the public SSH key
  key_name = aws_key_pair.mykeypair.key_name
  tags = {
    Name = "Yourname-ec2"
  }
}
```
save the file using "ESCAPE + :wq!"

Also, Create a new File for EC2 SecurityGroup
```
vi securitygroup.tf
```
Add the given lines, by pressing "INSERT" and replacing your name at `YourName-allow-ssh`
```
resource "aws_security_group" "allow-ssh" {
  vpc_id      = aws_vpc.main.id
  name        = "yourname-allow-ssh"
  description = "security group that allows ssh and all egress traffic"
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "YourName-allow-ssh"
  }
}
```
Save the file using "ESCAPE + :wq!"

Now, Create a new keypair that we use for our EC2 Instance.
```
ssh-keygen -f mykey
```
```
ls
```
```
vi key.tf
```
Add the given lines, by pressing "INSERT" and replace with `your name`.
```
resource "aws_key_pair" "mykeypair" {
  key_name   = "YourName-KeyPair"
  public_key = file(var.PATH_TO_PUBLIC_KEY)
}
```
Save the file using "ESCAPE + :wq!"
```
vi vars.tf
```
Add the given lines, by pressing "INSERT" and replace your `region` and add your `Region's AMI` to the list.

##### The first 3 lines are already present. Add the remaining lines below it.
```
variable "PATH_TO_PUBLIC_KEY" {
  default = "mykey.pub"
}

variable "AMIS" {
  type = map(string)
  default = {
    us-east-1 = "ami-0230bd60aa48260c6"
    us-west-2 = "ami-0a7d051a1c4b54f65"
    eu-west-1 = "ami-04c58523038d79132"
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
terraform plan
```
```
terraform apply -auto-approve
```
Once applied, verify that the `EC2 Instance` is created in the `Custom VPC` that we created in Task-1.

Also, to See the `Public IP` of the newly created EC2 Instance use the below command.
```
terraform state show aws_instance.example | grep public_ip
```
To `SSH` into newly launched EC2, run below command. Use either of the below depending on which OS you have used.
```
#ssh -i mykey ubuntu@<Your Public IP>
ssh -i mykey ec2-user@<Your Public IP>
```
Once verified, `Exit` from Instance.

### Task-3: Connecting an EBS with EC2 Instance 
```
vi instance.tf 
```
Add the given lines, by pressing "INSERT" The first block will be already present in the file. But, replace the entire code and update your `AvailabilityZone,` `YourName` in `AWS Instance` and `EBS Volume` both.
```
resource "aws_instance" "example" {
  ami = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  # the VPC subnet
  subnet_id = aws_subnet.main-public-1.id
  # the security group
  vpc_security_group_ids = [aws_security_group.allow-ssh.id]
  # the public SSH key
  key_name = aws_key_pair.mykeypair.key_name
  tags = {
    Name = "YourName-EC2"
  }
}

resource "aws_ebs_volume" "ebs-volume-1" {
 availability_zone = "us-east-1a"
 size = 20
 type = "gp2"
 tags = {
   Name = "Yourname extra volume"
 }
}

resource "aws_volume_attachment" "ebs-volume-1-attachment" {
  device_name = "/dev/xvdh"
  volume_id = aws_ebs_volume.ebs-volume-1.id
  instance_id = aws_instance.example.id
} 
```
Save the file using "ESCAPE + :wq!"
```
terraform apply
```
* Go to `EC2 Dashboard` and select the `EC2 Instance.` 
* Then select `storage` and see that the new volume is attached.

Once Done, Use the `terraform destroy` command to clean the infrastructure used in this lab.
```
terraform destroy -auto-approve
```
**Note:** Ensure to crosscheck that all the resources are destroyed or else it will charge you. 

Remove the directory and Zip file using `rm -rf`
```
cd ..
rm -rf lab_vpc
```
