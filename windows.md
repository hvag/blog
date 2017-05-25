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

## Remote State

When we build the VPCs, we have the build process generate output.  That output is stored in the VPC state file.  From other systems in the project, we can pull those outputs as inputs for use.  When creating the instance above, we query the VPC state file for:
- The id of the first public subnet
- The id of the default public security group
- The name of the project

So basically, we are spinning up a new Windows instance using a specified AMI and instance type.  We're placing it in our EAST DC attached to the first PUBLIC subnet.  We're configuring its virtual firewall via the default public security group.  And we are specifying the SSH key used for gaining access to the server.

## Configure Base Image

Now, that we have a running windows instance.  Let's go ahead and configure it to meet our baseline requirements.

Log into the server via RDP and perform the following:

1. Create a new account to be the Local Administrator and add this account to the Administrators group.  We will use this account for any subsequent activity.  All future servers built from this image will have this account/password included.

2. Run the EC2LaunchSettings application located in C:\ProgramData\Amazon\EC2-Windows\Launch\Settings and select `Shutdown with Sysprep`


## Create Base Image AMI

### instance-windows-image.tf (_updated_)

```
# Create Windows 2016 Server Instance
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
        Description = "Windows 2016 Golden Image - Source"
    }
}


resource "aws_ami_from_instance" "windows2016-Image" {
    name               = "windows2016-Image"
    source_instance_id = "${aws_instance.windows2016-Image.id}"

    tags {
        Name        = "windows2016-Image"
        Project     = "${data.terraform_remote_state.vpc-state.project-name}"
        Terraform   = "true"
        Description = "Windows 2016 Golden Image - AMI"
    }
}
```

## Create Base Image AMI (Encrypted)

Okay, it's a good thing we've got [InfoSec](/blog/security) engaged as part of the team.  By policy, we will be required to deliver encryption of data at rest or seek an exemption.  Let's give it a shot.

We can make a copy of the existing AMI, encrypting the copy in the process.  New instances built from the encrypted AMI will have an encrypted boot volume.

```
resource "aws_ami_copy" "windows2016-Image" {
    name              = "windows2016-Image-Encrypted"
    description       = "Windows 2016 Golden Image - AMI - Encrypted"
    source_ami_id     = "${aws_ami_from_instance.windows2016-Image.id}"
    source_ami_region = "${data.terraform_remote_state.vpc-state.east-region}"
    encrypted         = "true"

    tags {
        Name        = "windows2016-Image"
        Project     = "${data.terraform_remote_state.vpc-state.project-name}"
        Terraform   = "true"
        Description = "Windows 2016 Golden Image - AMI - Encrypted"
    }
}
```

## parked Items

Let's add status review for the original unencrypted instance and AMI to our [Parked Items](/blog/parkedItems)

### [Parked Items](/blog/parkedItems)

...
