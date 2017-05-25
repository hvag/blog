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

