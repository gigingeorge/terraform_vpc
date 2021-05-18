# Creating VPC using Terraform

Terraform is a tool for building infrastructure with various technologies including Amazon AWS, Microsoft Azure, Google Cloud, and vSphere.

Here is a simple document on how to use Terraform to build an AWS VPC along with private/public Subnet and Network ACL for the VPC. 
We will be making 1 VPC with 6 Subnets: 3 Private and 3 Public, 1 NAT Gateways, 1 Internet Gateway, and 2 Route Tables.
## Features

- Easy to customise and use as the Terraform modules are created using variables,allowing the module to be customized without altering the module's own source code, and allowing modules to be shared between different configurations.
- Each subnet CIDR block created  automatically using cidrsubnet Function 
- AWS informations are defined using tfvars file and can easily changed
- Project name is appended to the resources that are creating which will make easier to identify the resources. 

## Terraform Installation
- Create an IAM user on your AWS console that have access to create the required resources.
- Create a dedicated directory where you can create terraform configuration files.
- Download Terrafom, click here [Terraform](https://www.terraform.io/downloads.html).
- Install Terraform, click here [Terraform installation](https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started)

Use the following command to install Terraform
```sh
wget https://releases.hashicorp.com/terraform/0.15.3/terraform_0.15.3_linux_amd64.zip
unzip terraform_0.15.3_linux_amd64.zip 
ls -l
-rwxr-xr-x 1 root root 79991413 May  6 18:03 terraform  <<=======
-rw-r--r-- 1 root root 32743141 May  6 18:50 terraform_0.15.3_linux_amd64.zip
mv terraform /usr/bin/
which terraform 
/usr/bin/terraform
```
#### Lets create a file for declaring the variables. 
> Note : The terrafom files must be created with .tf extension. 

This is used to declare the variable and pass values to terraform source code.
```sh
vim variable.tf
```
#### Declare the variables for initialising terraform (for terraform provider file )
```sh
variable "region" {}
variable "access_key" {}
variable "secret_key" {}
variable "project" {}
variable "vpc_cidr" {}
```
#### Create the provider file
> Note : Terraform relies on plugins called "providers" to interact with remote systems. Terraform configurations must declare which providers they require, so that Terraform can install and use them. 
I'm using AWS as provider


```sh
vim provider.tf
```
```sh
provider "aws" {
  region     = var.region
  access_key = var.access_key
  secret_key = var.secret_key
}
```

#### Create a terraform.tfvars
> Note : A terraform.tfvars file is used to set the actual values of the variables.

```sh
vim terraform.tfvars
```
```sh
region = "< Desired region>"
access_key = "IAM user access_key"
secret_key = "IAM user secret_key"
project = "<project name>"
vpc_cidr = "<VPC cidr block>"
```
The Basic configuration for terraform aws is completed. Now we need to initialize the terraform using the loaded values
#### Terraform initialisation
```sh
terraform  init
```
> Once the initialization completes, the terraform is will able to connect to our AWS console as per the privileges set on IAM user on the defined region. 

Now we are going to create a VPC with 3 public subnets and 3 private subnets with all it's dependancies.
#### Create new VPC
```sh
vim main.tf
```
```sh
data "aws_availability_zones" "AZ" {
  state = "available"
}
```
> This will gather all the availability zones within our region.

#### To create VPC
```sh
resource "aws_vpc" "vpc" {
    
  cidr_block           = var.vpc_cidr
  instance_tenancy     = "default"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name = "${var.project}-vpc"
  }
}
```
 > Note : the format is  resource "resource_name" "local-resource-identifier" {}
 
 > local resource identfier is the name in which the deatils of defined resource is stored on tfstate file
 > tfstate file : This state is used by Terraform to map real world resources to your configuration, keep track of metadata, and to improve performance for large infrastructures. This state is stored by default in a local file named "terraform. tfstate"

#### To create InterGateWay For VPC
```sh
resource "aws_internet_gateway" "igw" {
    
  vpc_id = aws_vpc.vpc.id
  tags = {
    Name = "${var.project}-igw"
  }
}
```
> Note : vpc_id = aws_vpc.vpc.id  >> this refers to ID of VPC create with name "vpc"

Now we need to create Private and Public subnets. 

> PUBLIC SUBNET :  If a subnet's traffic is routed to an internet gateway, the subnet is known as a public subnet. 
> PRIVATE SUBNET : If a subnet doesn't have a route to the internet gateway, the subnet is known as a private subnet.

#### Creating public subnet
```sh
resource "aws_subnet" "public1" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 0)
  availability_zone        = element(data.aws_availability_zones.AZ.names,0)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-public1"
  }
}
```
```sh
resource "aws_subnet" "public2" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 1)
  availability_zone        = element(data.aws_availability_zones.AZ.names,1)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-public2"
  }
}
```
```sh
resource "aws_subnet" "public3" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 2)
  availability_zone        = element(data.aws_availability_zones.AZ.names,2)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-public-3"
  }
}
```
#### Creating Private subnet
```sh
resource "aws_subnet" "private1" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 3)
  availability_zone        = element(data.aws_availability_zones.AZ.names,0)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-private1"
  }
}
```
```sh
resource "aws_subnet" "private2" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 4)
  availability_zone        = element(data.aws_availability_zones.AZ.names,1)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-private2"
  }
}
```
```sh
resource "aws_subnet" "private3" {
    
  vpc_id                   = aws_vpc.vpc.id
  cidr_block               = cidrsubnet(var.vpc_cidr, 3, 5)
  availability_zone        = element(data.aws_availability_zones.AZ.names,2)
  map_public_ip_on_launch  = true
  tags = {
    Name = "${var.project}-private3"
  }
}
```

> cidrsubnet calculates a subnet address within given IP network address prefix.
```sh
 cidrsubnet(prefix, newbits, netnum) 
```
>   -  prefix must be given in CIDR notation
>   - newbits is the number of additional bits with which to extend the prefix. For example, if given a prefix ending in /16 and a newbits value of 4, the resulting subnet address will have length /20.
>    - netnum is a whole number that can be represented as a binary integer with no more than newbits binary digits, which will be used to populate the additional bits added to the prefix.

> element retrieves a single element from a list.
```sh
 element(list, index)
```
  - The index is zero-based. This function produces an error if used with an empty list. The index must be a non-negative integer.

Inorder for Private subnet to have internet connection, we need to configure NAT gateway. 

#### Create NAT gateway
> Note : It's required to have elastic IP to configure NAT gateway. 
> Nat gateway must be created on public subnet

###### Creating Elastic IP 

```sh
resource "aws_eip" "eip" {
  vpc      = true
  tags     = {
    Name = "${var.project}-eip"
  }
}
```
##### Assigning Elasic IP to Nat gateway

```sh
resource "aws_nat_gateway" "nat" {
    
  allocation_id = aws_eip.eip.id
  subnet_id     = aws_subnet.public1.id

  tags = {
    Name = "${var.project}-nat"
  }
}
```
Now we need to create route tables. 
A route table contains a set of rules, called routes, that are used to determine where network traffic from your subnet or gateway is directed.
- Each subnet in your VPC must be associated with a route table, which controls the routing for the subnet (subnet route table).
- A subnet can only be associated with one route table at a time, but you can associate multiple subnets with the same subnet route table.
- Every route table contains a local route for communication within the VPC.

> The route table for Public subnet should be assigned to interget gatway and for Private subnets it should be assigned with Nat gateway

#### Creating Route table 
###### For Public Subnets
```sh
resource "aws_route_table" "public" {
    
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "${var.project}-public"
  }
}
```
###### For Private Subnets
```sh
resource "aws_route_table" "private" {
    
  vpc_id = aws_vpc.vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat.id
  }

  tags = {
    Name = "${var.project}-private"
  }
}
```
The created Route tables must be associated to Public and Private Subnets
#### Route Table association
> Note : All the Public Subnets should be associated with Public Route table (via interget gateway)
```sh
resource "aws_route_table_association" "public1" {        
  subnet_id      = aws_subnet.public1.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public2" {      
  subnet_id      = aws_subnet.public2.id
  route_table_id = aws_route_table.public.id
}


resource "aws_route_table_association" "public3" {       
  subnet_id      = aws_subnet.public3.id
  route_table_id = aws_route_table.public.id
}
```
> Note : All the Privet Subnets should be associated with Private Route table (via Nat gateway)
```sh
resource "aws_route_table_association" "private1" {        
  subnet_id      = aws_subnet.private1.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private2" {      
  subnet_id      = aws_subnet.private2.id
  route_table_id = aws_route_table.private.id
}


resource "aws_route_table_association" "private3" {       
  subnet_id      = aws_subnet.private3.id
  route_table_id = aws_route_table.private.id
}
```

The VPC creation has been completed. 

Now we need the output of created resources 

#### Creating output variables
```sh
output "Elastic_IP" {
value = aws_eip.eip.public_ip
}
output "VPC" {
value = aws_vpc.vpc.id
}
output "Internet_Gateway" {
value = aws_internet_gateway.igw.id
}
output "Nat_Gatway" {
value = aws_nat_gateway.nat.id
}
output "Public_Route_table" {
value = aws_route_table.public.id
}
output "Private_Route_table" {
value = aws_route_table.private.id
}
```
#### Terraform Validation

> This will check for any errors on the source code
```sh
terraform validate
```
#### Terraform Plan

> The terraform plan command provides a preview of the actions that Terraform will take in order to configure resources per the configuration file. 
```sh
terraform plan
```
#### Terraform apply

> This will execute the tf file we created
```sh
terraform apply
```

