---
layout: page
title: MEAN
permalink: /MEAN/
published: true
---

Let's build a system utilizing the MEAN stack

First item - Deploy MongoDB tools to client (Mac).  Guideline for installing can be found [here:](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/){:target="_blank"}

- Download latest .tgz file
- Extract with 'tar'
- Move mongodb-osx-x86_64-3.4.6 directory to /usr/local/bin
- run 'ln -s /usr/local/bin/mongodb-osx-x86_64-3.4.6/bin/mongo /usr/local/bin/mongo'.  Mongo shell is now available.

Let's utilize Docker as part of this project.  Assuming Docker is already installed on the system...

```sudo docker run -d -p 27017:27017 -v <some_location>/data:/data/db mongo```







### Contact

[advisor@hvadvisorygroup.com](mailto:advisor@hvadvisorygroup.com)
