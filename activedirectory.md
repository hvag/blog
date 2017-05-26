---
layout: page-disqus
title: Active Directory
permalink: /activedirectory/
published: true
comments: true
---

Okay, we can now start building the Domain Controllers.

### instance-windows-DCs.tf

```
# Create Windows 2016 Server Instance
resource "aws_instance" "East-DC1" {
    ami = "${data.terraform_remote_state.east-state.windows2016-Image}"
    instance_type = "t2.micro"

    # VPC subnet
    subnet_id = "${element(split(":", data.terraform_remote_state.vpc-state.vpc-east-priv1-subnet-ids), 0)}"

    # Security Group
    vpc_security_group_ids = ["${data.terraform_remote_state.vpc-state.vpc-east-SG-PublicToPrivate-id}"]

    # Public SSH key
    key_name = "${data.terraform_remote_state.east-state.east-key-pair-id}"

    # Private IP - static
    private_ip = "10.100.0.10"

    root_block_device {
        volume_size = "40"
    }

    tags {
        Name      = "East-DC1"
        Project   = "${data.terraform_remote_state.vpc-state.project-name}"
        Terraform = "true"
        Description = "Domain Controller - DC1"
    }
}
```

For the DCs, we have create a new state file.  We are now three levels deep.


...
