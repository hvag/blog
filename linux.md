---
layout: page-disqus
title: Linux
permalink: /linux/
published: true
comments: true
---

## Building Linux Servers

First, we will create an instance to be used as the template for subsequent Linux servers.

### instance-linux-image.tf

```
# Create Ubuntu Server Instance
resource "aws_instance" "Ubuntu1604-Image" {
    ami = "ami-80861296"
    instance_type = "t2.micro"

    # VPC subnet
    subnet_id = "${element(split(":", data.terraform_remote_state.vpc-state.vpc-east-pub-subnet-ids), 0)}"

    # Security Group
    vpc_security_group_ids = ["${data.terraform_remote_state.vpc-state.vpc-east-SG-Public-Default-Linux-id}"]

    # Public SSH key
    key_name = "${aws_key_pair.TF-Demo-Dev-Key.key_name}"

    tags {
        Name      = "Ubuntu1604-Image"
        Project   = "${data.terraform_remote_state.vpc-state.project-name}"
        Terraform = "true"
        Description = "Ubuntu 16.04 2016 Golden Image - Source"
    }
}


resource "aws_ami_from_instance" "Ubuntu1604-Image" {
    name               = "Ubuntu1604-Image"
    source_instance_id = "${aws_instance.Ubuntu1604-Image.id}"

    tags {
        Name        = "Ubuntu1604-Image"
        Project     = "${data.terraform_remote_state.vpc-state.project-name}"
        Terraform   = "true"
        Description = "Ubuntu 16.04 2016 Golden Image - AMI"
    }
}


resource "aws_ami_copy" "Ubuntu1604-Image" {
    name              = "Ubuntu1604-Image-Encrypted"
    description       = "Ubuntu 16.04 2016 Golden Image - AMI - Encrypted"
    source_ami_id     = "${aws_ami_from_instance.Ubuntu1604-Image.id}"
    source_ami_region = "${data.terraform_remote_state.vpc-state.east-region}"
    encrypted         = "true"

    tags {
        Name        = "Ubuntu1604-Image"
        Project     = "${data.terraform_remote_state.vpc-state.project-name}"
        Terraform   = "true"
        Description = "Ubuntu 16.04 2016 Golden Image - AMI - Encrypted"
    }
}
```


## Bastion Host

### instance-linux-Bastion.tf

```
# Create Windows 2016 Server Instance
resource "aws_instance" "East-Bastion-L1" {
    ami = "${aws_ami_copy.Ubuntu1604-Image.id}"
    instance_type = "t2.micro"

    # VPC subnet
    subnet_id = "${element(split(":", data.terraform_remote_state.vpc-state.vpc-east-pub-subnet-ids), 0)}"

    # Security Group
    vpc_security_group_ids = ["${data.terraform_remote_state.vpc-state.vpc-east-SG-Public-Default-Linux-id}"]

    # Public SSH key
    key_name = "${aws_key_pair.TF-Demo-Dev-Key.key_name}"

    # Private IP - static
    private_ip = "10.100.128.11"

    tags {
        Name        = "East-Bastion-L1"
        Project     = "${data.terraform_remote_state.vpc-state.project-name}"
        Terraform   = "true"
        Description = "Linux Bastion Host"
    }
}
```

### [Parked Items](/blog/parkedItems)

