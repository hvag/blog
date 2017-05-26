---
layout: page-disqus
title: Active Directory
permalink: /activedirectory/
published: true
comments: true
---

Okay, we can now start building the Domain Controllers.

**Security Note** - Let's isolate the AD and other control servers to the PRIV2 subnet (security boundary).  We will initially allow all traffic between the servers on this subnet.  Additional lock-down is pending.

## First DC

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

For the DCs, we have created a new state file.  We are now three levels deep.  We will perform a few manual activities.

Also, would like increased performance for this server so will switch instance_type from t2.micro to t2.small. 

1. In Server Manager, change the name of the server from the AWS generated name to EAST-DC1
1. Configure the Network Device.  For TCP/IPv4 set the Preferred DNS server to 127.0.0.1
1. Disable native firewall - Add to parking lot for final config.


## Second DC

DC1 and DC2 have been created in separate subnets/availability zones/rooms.  They will not both be impacted by any local event.  We will eventually add DCs to the WEST data center to account for any insanely large geographic impacting event.

### [Parked Items](/blog/parkedItems)

...
