---
layout: page-disqus
title: Windows
permalink: /windows/
published: true
comments: true
---

## Building Windows Servers

First, we will create an instance to be used as the template for subsequent Windows servers.  Is it wrong to knock this out using the AWS console?  Feels wrong, but just a minor detour.

Okay, it's wrong.  Let's deploy the 'image' instance via Terraform.

### main.tf
```
# During build, the VPC outputs the region that it is constructed in.
# Get that for provider region below.

data "terraform_remote_state" "vpc-state" {
    backend = "s3"
    config {
        bucket  = "hvag-tfdemo-state"
        key     = "vpc/terraform.tfstate"
        region  = "us-east-1"
    }
}


provider "aws" {
    region = "${data.terraform_remote_state.vpc-state.east-region}"
}


# Query AWS for the latest Windows 2016 Base AMI
# We don't yet want the image to be rebuilt whenever there's a new AMI
# We will use the current latest: ami-f1b5cfe7
data "aws_ami" "windows2016-image" {
    most_recent = "true"

    filter {
    name   = "name"
    values = ["Windows_Server-2016-English-Full-Base*"]
  }
}
```

### instance-windows-image.tf
```
resource "aws_instance" "windows2016-Image" {
    ami = "ami-f1b5cfe7"
    instance_type = "t2.micro"

    # VPC subnet
    subnet_id = "${element(split(":", data.terraform_remote_state.vpc-state.vpc-east-pub-subnet-ids), 0)}"

    # Security Group
    vpc_security_group_ids = ["${data.terraform_remote_state.vpc-state.vpc-east-SG-Public-Default-Win-id}"]

    # Public SSH key
    key_name = "${aws_key_pair.TF-Demo-Dev-Key.key_name}"

    tags {
        Name      = "windows2016-Image"
        Project   = "${data.terraform_remote_state.vpc-state.project-name}"
        Terraform = "true"
    }
}
```

Not only did we use Terraform for creating the instance, but we got to try something new.  Notice the usage of `terraform_remote_state`.

When we build the VPCs, we have the build process generate output.  That output is stored in the VPC state file.  From other systems in the project, we can pull those outputs as inputs for use.  When creating the instance above, we query the VPC state file for:
- the id of the first public subnet
- the id of the default public security group
- the name of the project

So basically, we are spinning up a new Windows instance with a specified AMI and instance type.  We're placing it in our EAST DC attached to the first PUBLIC subnet.  We're configuring its virtual firewall via the default public security group.  And we are specifying the SSH key used for gaining access to the server.


...

