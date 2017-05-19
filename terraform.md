---
layout: page-disqus
title: Terraform
permalink: /terraform/
published: true
comments: true
---

Let's use Terraform to create a bit of new infrastructure - This doc is a work in progress.  Check back for updates.  Follow progress on twitter: 
[@_markshaw](https://twitter.com/_markshaw){:target="_blank"}

-
### 1. Purpose

An example of utilizing Terraform to build components

-
### 2. Scope

Infrastructure - concept to build via Terraform

-
### 3. References

NA

-
### 4. Definitions

NA

-
### 5. Responsibilities

NA

-
### 6. Materials and Equipment

  6.1 An existing AWS account
  
  6.2 The AWS CLI is installed
  
  6.3 Terraform v0.9.5 or above is installed

-
### 7. Make it so

Let's start our build with AWS.  Why AWS?  Why not AWS; besides, AWS is the space where confidence is high that we can produce a quick win.

I open my eyes.  I'm standing in, a void.  Not darkness, light, like a completely clean sheet of paper.  A cube, a sphere, a cloud, unknown, the edges are unseen, they extend beyond.

I need something.  I need my own space within the space.  A VPC.

* Open a terminal session
  
* Create a new working directory to contain the Terraform files.  Oh, wait a minute.  No, let's set up a GitHub repository, we're going to want to share and have version control.  I create a new GitHub repository tf-demo, it's Public and Initialized with a README.  Note to self, **do not add any 'secrets' to this repo**.

  :bulb: **Tip**:  Highly recommend, protect both your AWS and GitHub accounts with two-factor authentication.  Actually, you ought to protect all your accounts with two-factor authentication.

* Clone the repo:  
	`git clone https://hvag@github.com/hvag/tf-demo.git`
    
* Create a .gitignore file:

```
## .gitignore

.terraform
*.key
terraform.tfstate.*
```

I have my terminal session, I have my AWS account, I have the AWS CI installed.  Let's tie them together.  From within the tf-demo directory created via git clone, perform the following:

	$ export AWS_ACCESS_KEY_ID=**********
	$ export AWS_SECRET_ACCESS_KEY=********************
	$ export AWS_DEFAULT_REGION=us-east-1
    
Let's check status:

	$ aws ec2 describe-regions
    
Did you get a list of AWS regions?  Excellent.

## Infrastructure as Code
If only all infrastructure could be code.  Sure, it eventually has to run on something, but having your infrastructure just be a manifestation, an instantiation of your idea or ideal; isn't that interesting?

Interestingly enough, once you start down this path, you will eventually hit an inflection point at which it will be simpler to decipher infrastructure by examining the code vs. clicking through a console. As with everything, just takes a bit of practice. Yep, we’re talking ‘bout practice!

## VPC

Initial plan was to create a VPC. A, as in singular.  Yes, a region is made up of multiple availability zones, but hey, we're conceptualizing, why not have the capability to have redundancy via different geographic regions?  I say we build one VPC on the east coast and one VPC on the west coast.  We can revisit later to discuss being globally distributed.  Think about it this way, we're going to construct infrastructure comprised of a Data Center (DC) in northern Virginia and a DC in northern California.  For redundancy, each DC will be comprised of multiple physical rooms spread across large campuses.  Each room will be independent, highly redundant and connected on each campus via a high-speed network.

## Terraform State

Terraform utilizes state files to track the current state of the deployed infrastructure. The state file is a JSON representation of all the objects currently under Terraform control. Terraform allows us to utilize a ‘backend’ store to keep the state files in a centralized location and accessible to multiple developers. For our demonstration, we will utilize AWS S3 as the backend.


1. Create a S3 bucket as a container for the Terraform state files and enable versioning.  

	```
    aws s3api create-bucket --bucket hvag-tfdemo-state --region us-east-1
    
    aws s3api put-bucket-versioning --bucket hvag-tfdemo-state --versioning-configuration Status=Enabled
    ```

2. Create a file **s3-backend.tf** in the tf-demo folder.

	```ruby
    terraform {
        backend "s3" {
            bucket  = "hvag-tfdemo-state"
            region  = "us-east-1"
            key     = "terraform.tfstate"
            encrypt = "true"
        }
    }
    ```
    
3. Run ```terraform init``` to configure the backend for use.


## DRY - Don't repeat yourself

Yikes, one of the paradigms of coding (_at least I think it's a paradigm, let's go with it_) is that one ought to not repeat one's self.  I'm creating two VPCs but I really ought to code it just once.


## Terraform Module

To achieve code reusability, we will utilize Terraform modules.

- Create a 'modules' subdirectory as a container for the modules.  Create a vpc subdirectory in modules.  Our directory now looks like this:

```
|____modules
| |____vpc
| | |____module-vpc.tf
| | |____outputs.tf
| | |____vars.tf
```

#### vars.tf

```ruby
variable "region" {
    description = "In which AWS region should the VPC be created"
}

variable "name" {
    description = "Name of the VPC"
}

variable "cidr_block" {
    description = "VPC CIDR"
}
```

#### module-vpc.tf

```ruby
provider "aws" {
      region = "${var.region}"
}

# Create VPC
resource "aws_vpc" "TF-DEMO" {
    cidr_block = "${var.cidr_block}"

    instance_tenancy = "default"
    enable_dns_support = true
    enable_dns_hostnames = true

    tags {
        Name = "${var.name}"
        terraform = "true"
    }
}
```

#### outputs.tf

```ruby
output "vpc_name" {
    value = "${aws_vpc.TF-DEMO.name}"
}
```

- Create the following file in the tf-demo directory

#### vpc-tf

```ruby
module "vpc-east" {
    source = "modules/vpc"

    region     = "us-east-1"
    name       = "VPC-EAST"
    cidr_block = "10.100.0.0/16"
}

module "vpc-west" {
    source = "modules/vpc"

    region     = "us-west-1"
    name       = "VPC-WEST"
    cidr_block = "10.200.0.0/16"
}
```

- Now perform the following:

```ruby
terraform get		<- Get all modules
terraform plan		<- Show the execution plan
terraform apply		<- Build or change infrastructure
```

And with that, we have completed the first item in our backlog.  We have code that will "present" two geographically distributed data centers.

And, speaking of plan, would now be the time to step back and produce an overall design for the desired end-state?  I say no, that would be so "waterfall-ly".  Let's take a stab at agile. We'll plan short sprints with frequent builds, generating and working from user stories.

Just because we can, lets run `terraform destroy` to blow-up the DCs.  We can run `terraform apply` at any time to recreate them.  And, with destruction being so easy, lets add user stories regarding controls and recovery capabilities.

## VPC Network
_Hey, those are nice new buildings we have; can we put computers in them?_  
_No, we need more stuff_

From the VPC buildout, each campus has been assigned a 10.X.0.0/16 network.  There are currently up to four DC rooms on each campus, designated R1 - R4 with subnets as shown below for VPC-EAST. Each room will be configured to have:
- 1 Public facing subnet
- 2 Private subnets
- Reserved capacity

Subnet            | Address Ranges                  | Name
:---------------- | :------------------------------ | :-------------
10.100.0.0/19     | 10.100.0.1   - 10.100.31.254    | VPC-E-R1-Priv1
10.100.32.0/19    | 10.100.32.1  - 10.100.63.254    | VPC-E-R2-Priv1
10.100.64.0/19    | 10.100.64.1  - 10.100.95.254    | VPC-E-R3-Priv1
10.100.96.0/19    | 10.100.96.1  - 10.100.127.254   | VPC-E-R4-Priv1
10.100.128.0/20   | 10.100.128.1 - 10.100.143.254   | VPC-E-R1-Pub
10.100.144.0/20   | 10.100.144.1 - 10.100.159.254   | VPC-E-R2-Pub
10.100.160.0/20   | 10.100.160.1 - 10.100.175.254   | VPC-E-R3-Pub
10.100.176.0/20   | 10.100.176.1 - 10.100.191.254   | VPC-E-R4-Pub
10.100.192.0/21   | 10.100.192.1 - 10.100.199.254   | VPC-E-R1-Priv2
10.100.200.0/21   | 10.100.200.1 - 10.100.207.254   | VPC-E-R2-Priv2
10.100.208.0/21   | 10.100.208.1 - 10.100.215.254   | VPC-E-R3-Priv2
10.100.216.0/21   | 10.100.216.1 - 10.100.223.254   | VPC-E-R4-Priv2
10.100.224.0/21   | 10.100.224.1 - 10.100.231.254   | VPC-E-R1-Spare
10.100.232.0/21   | 10.100.232.1 - 10.100.239.254   | VPC-E-R2-Spare
10.100.240.0/21   | 10.100.240.1 - 10.100.247.254   | VPC-E-R3-Spare
10.100.248.0/21   | 10.100.248.1 - 10.100.255.254   | VPC-E-R4-Spare


### 2017.05.19 Network is in place

Added a new module and performed a bit of refactoring.  Code located [here](https://github.com/hvag/tf-demo){:target="_blank"}

Directory structure now looks like this:

```
|____main-vars.tf
|____main.tf
|____modules
| |____vpc
| | |____module-vpc.tf
| | |____outputs.tf
| | |____vars.tf
| |____vpc-subnets
| | |____module-subnets.tf
| | |____vars.tf
|____README.md
|____s3-backend.tf
```

#### main-vars.tf
```
variable east-region { default = "us-east-2" }
variable west-region { default = "us-west-2" }

variable vpc-east-network-address { default = "10.100.0.0/16" }
variable vpc-west-network-address { default = "10.200.0.0/16" } 

variable vpc-east-network-subnet { default = "10.100" }
variable vpc-west-network-subnet { default = "10.200" }
```

Here we have a few bootstrap configuration variables.

- In which AWS regions will we create the VPCs
- What are the network addresses for the VPCs
- Prefixes for the VPC subnets

### main.tf

```
module "vpc-east" {
    source = "modules/vpc"

    project-name    = "TF-DEMO"
    name            = "VPC-EAST"
    region          = "${var.east-region}"
    network-address = "${var.vpc-east-network-address}"
}

module "vpc-west" {
    source = "modules/vpc"

    project-name    = "TF-DEMO"
    name            = "VPC-WEST"
    region          = "${var.west-region}"
    network-address = "${var.vpc-west-network-address}"
}

module "vpc-east-subnets" {
    source = "modules/vpc-subnets"

    vpc-id                 = "${module.vpc-east.vpc-id}"

    project-name           = "TF-DEMO"
    region                 = "${var.east-region}"
    priv1_subnet_addresses = [ "${var.vpc-east-network-subnet}.0.0/19",   "${var.vpc-east-network-subnet}.32.0/19",  "${var.vpc-east-network-subnet}.64.0/19"  ]
    priv2_subnet_addresses = [ "${var.vpc-east-network-subnet}.192.0/21", "${var.vpc-east-network-subnet}.200.0/21", "${var.vpc-east-network-subnet}.208.0/21" ]
    pub_subnet_addresses   = [ "${var.vpc-east-network-subnet}.128.0/20", "${var.vpc-east-network-subnet}.144.0/20", "${var.vpc-east-network-subnet}.160.0/20" ]
}

module "vpc-west-subnets" {
    source = "modules/vpc-subnets"

    vpc-id                 = "${module.vpc-west.vpc-id}"

    project-name           = "TF-DEMO"
    region                 = "${var.west-region}"
    priv1_subnet_addresses = ["${var.vpc-west-network-subnet}.0.0/19",   "${var.vpc-west-network-subnet}.32.0/19",  "${var.vpc-west-network-subnet}.64.0/19"  ]
    priv2_subnet_addresses = ["${var.vpc-west-network-subnet}.192.0/21", "${var.vpc-west-network-subnet}.200.0/21", "${var.vpc-west-network-subnet}.208.0/21" ]
    pub_subnet_addresses   = ["${var.vpc-west-network-subnet}.128.0/20", "${var.vpc-west-network-subnet}.144.0/20", "${var.vpc-west-network-subnet}.160.0/20" ]
}
```

- Call the VPC module to build VPC-EAST
- Call the VPC module to build VPC-WEST
- Call the VPC-subnets module to build VPC-EAST-subnets
- Call the VPC-subnets module to build VPC-WEST-subnets

Each "campus" currently has three "rooms".  If a fourth room is constructed a network segment can be added as shown in the VPN network layout above.  Also, the original Virginia and California DCs were torn-down and reconstructed in Ohio and Oregon.  How crazy is that?

### module-vpc.tf

```
provider "aws" {
      region = "${var.region}"
}

# Create VPC
resource "aws_vpc" "vpc" {
    cidr_block = "${var.network-address}"

    instance_tenancy = "default"
    enable_dns_support = true
    enable_dns_hostnames = true

    tags {
        Name      = "${var.name}"
        Project   = "${var.project-name}"
        Terraform = "true"
    }
}
```

### module-subnets.tf

```
provider "aws" {
      region = "${var.region}"
}

# Query AWS data source for list of available AZs
data "aws_availability_zones" "available-azs" {}

# Create Subnets utilizing priv1_subnet_addresses list
resource "aws_subnet" "Priv1" {
    vpc_id                  = "${var.vpc-id}"
    count                   = "${length(var.priv1_subnet_addresses)}"
    cidr_block              = "${element(var.priv1_subnet_addresses, count.index)}"
    availability_zone       = "${element(data.aws_availability_zones.available-azs.names, count.index)}"
    map_public_ip_on_launch = false

    tags {
        Name      = "Priv1-${count.index}"
        Project   = "${var.project-name}"
        Terraform = "true"
    }
}

# Create Subnets utilizing priv2_subnet_addresses list
resource "aws_subnet" "Priv2" {
    vpc_id                  = "${var.vpc-id}"
    count                   = "${length(var.priv2_subnet_addresses)}"
    cidr_block              = "${element(var.priv2_subnet_addresses, count.index)}"
    availability_zone       = "${element(data.aws_availability_zones.available-azs.names, count.index)}"
    map_public_ip_on_launch = false

    tags {
        Name      = "Priv2-${count.index}"
        Project   = "${var.project-name}"
        Terraform = "true"
    }
}

# Create Subnets utilizing pub_subnet_addresses list
resource "aws_subnet" "Pub" {
    vpc_id                  = "${var.vpc-id}"
    count                   = "${length(var.pub_subnet_addresses)}"
    cidr_block              = "${element(var.pub_subnet_addresses, count.index)}"
    availability_zone       = "${element(data.aws_availability_zones.available-azs.names, count.index)}"
    map_public_ip_on_launch = true

    tags {
        Name      = "Pub-${count.index}"
        Project   = "${var.project-name}"
        Terraform = "true"
    }
}
```

Utilizing count and iterating over the injected subnet address lists makes it all so much easier and cleaner.

## Building Servers
_I heard that the network was ready, now can I build my server?_  
_Well, we're not quite there yet; but, the PM says that we can Fast Track!_


...
