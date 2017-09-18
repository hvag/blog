---
layout: page
title: itNow
permalink: /itNow/
published: true
---

## What we liked yesterday

 - [Windows](/blog/windows)
 - [Linux](/blog/linux)
 - [Active Directory](/blog/activedirectory)
 - Exchange
 - Oracle
 - VMware
 
 
## What we like right now

### Model
 - Microservices - Small reusable components (horizontal scaling)
 - Declarative Model - This is what I want, I won't tell you how to do it
 
### Tools
 - Node.js - Common language across stack
 - Swift - See Node.js
 - Docker - hmm, container.  It works on my computer, it will work on your also - No more application dependencies
 - Kubernetes - I can control you all, 'cause actually, I just have to control one machine, the kb machine, universal abstraction, nice!  Stop thinking about Java
 - Lambda - Hey, wait a minute, what if I didn't even have to build and orchestrate the containers?  What if I could just launch the app/service?  It would kinda be '**Serverless**' - for me anyway
 
### Testing
 - Mocha
 
Let's get started with our 'Hello World' express app
 
### Express App
 
```
const os = require("os")
const express = require('express')
const app = express()

const port = process.env.PORT || 3000
const response = "Hello World.  I'm being served from a Docker container!"

let server = null

app.get('/', (req, res) => {
    res.status(200).send(response)
})

app.get('/ni', (req, res) => {
    res.status(200).send(os.networkInterfaces())
})

function onListen() {
    const host = server.address().address
    const port = server.address().port

    console.log('App (server) listening at http://%s:%s', host, port)
}

if (module.parent) {
    module.exports = app
} else {
    server = app.listen(port, onListen)
}
```

### Test & Build

Let's see how much we can 'control' from our package.json

```
{
  "name": "myserver",
  "version": "0.0.1",
  "private": true,
  "description": "My Server",
  "main": "server.js",
  "scripts": {
    "test": "NODE_ENV=test ./node_modules/.bin/mocha --reporter spec --timeout 10000 || true",
    "start": "node server.js",
    "d-build": "docker build -t markshaw/express-demo:prod .",
    "d-build-test": "docker build -t markshaw/express-demo:test .",
    "d-run": "docker run -it -p 3000:3000 markshaw/express-demo:prod",
    "d-run-test": "docker run -it -p 3000:3000 markshaw/express-demo:test",
    "help": "echo 'Commands:\n\nhelp - show this message\ntest - Mocha test\nd-build - Docker build\nd-run - run local (Docker)\nstart - run local\n'"
  },
  "author": "Mark",
  "license": "ISC",
  "dependencies": {
    "express": "^4.15.4"
  },
  "devDependencies": {
    "chai": "^4.1.1",
    "chai-http": "^3.0.0",
    "mocha": "^3.5.0",
    "supertest": "^3.0.0"
  }
}
```
