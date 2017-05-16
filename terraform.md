---
layout: page
title: Terraform
permalink: /terraform/
published: true
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

* I clone the repo with: `git clone https://hvag@github.com/hvag/tf-demo.git`

I have my terminal session, I have my AWS account, I have the AWS CI installed.  Let's tie them together.

	$ export AWS_ACCESS_KEY_ID=**********
	$ export AWS_SECRET_ACCESS_KEY=********************
	$ export AWS_DEFAULT_REGION=us-east-1
    
Let's see if that worked

	$ aws ec2 describe-regions
    
Did you get a list of AWS regions?  Excellent.


