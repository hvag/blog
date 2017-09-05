---
layout: page
title: MEAN
permalink: /mean/
published: true
---
Let's build a system utilizing the MEAN stack

We need a 'container' that will store and mange arbitrary JavaScript objects (JSON documents).  Let's start with MongoDB but we'll plan to look at Firebase and DynamoDB.

### Deploy MongoDB tools to client (Mac).  Guideline for installing can be found [here:](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/){:target="_blank"}

- Download latest .tgz file
- Extract with 'tar'
- I choose to move the extracted 'bin' files to /usr/local/bin
- Mongo shell is now available.

### Setup MongoDB
Let's utilize Docker as part of this project.  Assuming Docker is already installed on the system...

```docker run --name mean-mongo -d -p 27017:27017 -v <some_location>/data:/data/db mongo```

run mongo client

Let's test DB
 - db.test.insert({ key: "value" })
 - db.test.find()

Excellent!  Mongo is up and running in a Docker container and using the specified host location as a volume for database storage.


### Node.js

We like JavaScript so much that we want to run it outside our browser; yeah [Node.js](https://nodejs.org).  We also like Swift + Kitura but let's get started with Node + Express.  Will I ever have time to take a look at Go?

Kitura and Express are [Web Application Frameworks](https://en.wikipedia.org/wiki/Web_framework)

Create a new folder for the Node application and run 'npm init' to configure the package.json file

Let's add a few dependencies and create the node_modules folder

**run 'npm install --save mocha mongoose express'.**


### Git

Let's create a new git repository - run 'git init'


### Mocha

Let's utilize Mocha for testing.  Let's also install node module [supertest](https://www.npmjs.com/package/supertest) so we can have a 'client' to make calls to our app during testing.

Here is an example of our App and associated test

**app.js**
```
const express = require('express')

const app = express()

/**
 * Callback:
 * @param {object} req The incoming request
 * @param {object} res The outgoing response
 */
app.get('/api', (req, res) => {
    res.send({ hello: 'World!' })
})

module.exports = app
```

**index.html**
```
const app = require('./app')

const PORT = process.env.PORT || 3000

app.listen(PORT, () => {
    console.log('Listening on PORT %s', PORT)
})
```

**app_test.js**
```
const assert = require('assert')
const request = require('supertest')

const app = require('../app')

describe('hvagGeneriCRUD Express App', () => {

    it('handles a GET request to /api', (done) => {
        request(app)
            .get('/api')
            .end((err, response) => {
                assert(response.body.hello === 'World!')
                done()
            })
    })

})
```

### Body-Parser

Express is not great for handling the {body} of a POST request.  For this functionality, we will use the [body-parser](https://www.npmjs.com/package/body-parser) library - Node.js body parsing middleware.  This will add a .body property to the incoming request.

Body-Parser was previously included with Express but is now a separate library.

**run 'npm install --save body-parser**


### Mongoose

Let's bring Mongoose into play.  Mongoose works with MongoDB, simplifying the modeling of application data.  Here is an example of Mongoose in action (from:[mongoosejs.com](http://mongoosejs.com)
```
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/test');

var Cat = mongoose.model('Cat', { name: String });

var kitty = new Cat({ name: 'Zildjian' });
kitty.save(function (err) {
  if (err) {
    console.log(err);
  } else {
    console.log('meow');
  }
});
```



...

### Contact

[advisor@hvadvisorygroup.com](mailto:advisor@hvadvisorygroup.com)