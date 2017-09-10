---
layout: page
title: Automation
permalink: /automation/
published: true
---

Let's attempt to create a CI/CD pipeline utilizing Jenkins

### GitHub

Create new [GitHub repo](https://github.com/hmashaw/docker-node-jenkins)

Clone repo

### Node

Create a functioning Node application

Commit and push application to repo

### Jenkins

Create a new Jenkins Project

- configure the project to pull code from the Github repo
- Add a single build step - 'npm install'


### Docker

For review - [Docker Security](https://www.projectatomic.io/blog/2015/08/why-we-dont-let-non-root-users-run-docker-in-centos-fedora-or-rhel/)

There are many Docker plugins available for Jenkins.  We will work with the 'CloudBees Docker Build and Publish' plugin