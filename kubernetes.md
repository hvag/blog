---
layout: page
title: Kubernetes
permalink: /kubernetes/
published: true
---

Container - A system process

Docker - Here are my strong opinions on how you should build containment around your system process so it can best be isolated from other running proceses and from the host

Image - I will not depend on the host, I will bring all my dependencies with me

Kubernetes - I've got a lot of images, I think I need a bit of help managing them.  Here's my desired state, please enforce it

You thought I was a SysAdmin.  Well, I was replaced by a script, sorta.  But, it's all good, k8s is way more interesting

Let's take a stab at setting up a Kubernetes cluster

| Item | AWS | GCP |
| ---- | --- | --- |
| Setup cluster | kops - Kubernetes Operations | gcloud |

### Configuring tools to start a k8s cluster on AWS

For this exercise, let's use an Ubuntu Workstation - 16.04 LTS

  * Install kubectl - [https://kubernetes.io/docs/tasks/tools/install-kubectl/](https://kubernetes.io/docs/tasks/tools/install-kubectl/){:target="_blank"}

  * Install kops - [https://github.com/kubernetes/kops](https://github.com/kubernetes/kops){:target="_blank"}
  
  * Install AWS CLI - [https://aws.amazon.com/cli/](https://aws.amazon.com/cli){:target="_blank"}
  
  * Generate ssh keys to be used when loggin into the cluster - will utilize **ssh-keygen**
  
On AWS:

  * Configure a IAM user for kops.  Will attach the following policies (via a group):
  
    * AmazonEC2FullAccess
    * IAMFullAccess
    * AmazonS3FullAccess
    * AmazonVPCFullAccess
    * AmazonRoute53FullAccess
    
  * Create a S3 bucket to contain kops state
  
  * Configure a kubernetes subdomain via Route 53.  We will use **kubernetes.domainName.com**
  
### Using kops to create the cluster

To preview the changes to be made on AWS:

```kops create cluster --name=<clusterName> --state=s3://<bucket-name> --zones=us-east-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=kubernetes.<domain>.com --ssh-public-key=<publicKey>```

#### Interesting Options
  * Build a terraform manifest with ```--target=terraform``` then use **terraform apply** to apply the configuration
  
  * Build a private network topology with ```--topology private```

Finally configure your cluster with: ```kops update cluster <clusterName> --yes```

To check the status of the nodes: ```kubectl get nodes```

And, to eventually delete the cluster: ```kops delete cluster --name=<clusterName>```


### Kebernetes

  * node - machine running kubelet
  * pod - n containers (logical unit running on a node)
  * scheduler - finds best fit (node) for pods
  * replication controller/deployment - templates for desired state
  * service - vip for pods (think load balancer)
  
  
  
  
  
  
  
  
  
  

