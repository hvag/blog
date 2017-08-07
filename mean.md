---
layout: page
title: MEAN
permalink: /MEAN/
published: true
---

Let's build a system utilizing the MEAN stack

We need a 'thing' that will store and mange arbitrary JavaScript objects (JSON documents).  Let's start with MongoDB but we'll plan to look at Firebase and DynamoDB.

### Deploy MongoDB tools to client (Mac).  Guideline for installing can be found [here:](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/){:target="_blank"}

- Download latest .tgz file
- Extract with 'tar'
- Move mongodb-osx-x86_64-3.4.6 directory to /usr/local/bin
- run 'ln -s /usr/local/bin/mongodb-osx-x86_64-3.4.6/bin/mongo /usr/local/bin/mongo'.  Mongo shell is now available.

### Setup MongoDB
Let's utilize Docker as part of this project.  Assuming Docker is already installed on the system...

```docker run --name mean-mongo -d -p 27017:27017 -v <some_location>/data:/data/db mongo```

Let's test DB (Mongo uses database named 'test' as default
 - db.test.insert({ key: "value" });
 - db.test.findOne({ key: "value" });

Excellent!  Mongo is up and running in a Docker container and using the specified host location as a volume for database storage.

## MongoDB Index Example

```
db.users.insert({ name: 'Tony' });
db.users.find({ name: 'Tony' });

db.users.createIndex({name: 1});
{
	"createdCollectionAutomatically" : false,
	"numIndexesBefore" : 1,
	"numIndexesAfter" : 2,
	"ok" : 1
}

```


### Node.js

Create a folder for the Node application and run 'npm init' to configure the package.json file

run 'npm install underscore --save'.  We will add our first dependency and create the node_modules folder


### Contact

[advisor@hvadvisorygroup.com](mailto:advisor@hvadvisorygroup.com)
