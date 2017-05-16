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

  Tip:  Highly recommend, protect both your AWS and GitHub accounts with two-factor authentication.  Actually, you ought to protect all your accounts with two-factor authentication.

* I clone the repo with: `git clone https://hvag@github.com/hvag/tf-demo.git`

I have my terminal session, I have my AWS account, I have the AWS CI installed.  Let's tie them together.  From within the tf-demo directory created via git clone, perform the following:

	$ export AWS_ACCESS_KEY_ID=**********
	$ export AWS_SECRET_ACCESS_KEY=********************
	$ export AWS_DEFAULT_REGION=us-east-1
    
Let's check status:

	$ aws ec2 describe-regions
    
Did you get a list of AWS regions?  Excellent.

## Infrastructure as Code
If only all infrastructure could be code.  Sure, it eventually has to run on something, but having your infrastructure just be a manifestation, an instantiation of your idea or ideal; isn't that interesting?

Interestingly enough, once you start down this path, you will eventually hit an inflection point at which it will be simplier to decipher infrastructure by examing the code vs. clicking through a console.  As with everything, just takes a bit of practice.

## VPC

Initial plan was to create a VPC. A, as in singular.  Yes, a region is made up of multiple availability zones, but hey, we're conceptualizing, why not have the capability to have redundancy via different geographic regions?  I say we build one VPC on the east coast and one VPC on the west coast.  We can revisit later to discuss being globally distributed.  Think about it this way, we're going to construct infrastructure comprised of a Data Center (DC) in northern Virginia and a DC in northern California.  For redundancy, each DC will be comprised of multiple physical rooms spread across large campuses.  Each room will be independent, highly redundant and connected on each campus via a high-speed network.


## DRY - Don't repeat yourself

Yikes, one of the paradigms of coding (_at least I think it's a paradigm, let's go with it_) is that one ought not repeat one's self.  I'm creating two VPCs but I really ought to code it just once.

Hmm, lemme think about that.  Remember, feel free to send me your ideas ...

...


