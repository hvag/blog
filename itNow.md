---
layout: page
title: itNow
permalink: /itNow/
published: true
---

What we love right now

### Model
 - Microservices - Small reusable components
 - Declarative Model - This is what I want, I won't tell you how to do it
 
### Tools
 - Node.js - It works on my computer, it will work on your also
 - Docker - hmm, container
 - Kubernetes - I can control you all
 
 ### My Express App
 
```
 /**
 * 
 */

const os = require("os")
const express = require('express')
const app = express()

app.get('/', (req, res) => {
    res.send("Hello World.  I'm being served from a Docker container!")
})

app.get('/ni', (req, res) => {
    res.send(os.networkInterfaces())
})

function onListen() {
    const host = server.address().address
    const port = server.address().port

    console.log('App (server) listening at http://%s:%s', host, port)
}

var server = app.listen(process.env.port || 3000, onListen)
```
