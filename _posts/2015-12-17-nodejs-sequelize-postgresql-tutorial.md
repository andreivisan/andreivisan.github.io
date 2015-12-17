---
layout: post
title: Node.js, Sequelize, PostgreSQL tutorial
published: false
---

### Introduction

In this simple tutorial I will show you how to setup and start using Node.js together with Sequelize and Postgres. These are very 
popular technologies, yet not so widely covered and properly explained how to use together. I will not cover the benefits of each of these technologies since they are very well documented independently. What I will do is show you how to use them together by creating the already infamous To Do List project.

You can find the full code for this tutorial on <a href="https://github.com/andreivisan/node-sequelize-postgresql" target="_blank"> Github </a>.

### Configuration

To start I recommend using <a href="http://postgresapp.com/" target="_blank"> Postgress.app </a> if you are a Mac user, or just install PostgreSQL if you use another OS.
Next, lets create a Node.js project and install the following modules to work with Sequelize and Postgres:

  ```
  npm install sequelize --save
  npm install pg --save
  npm install pg-hstore --save
  ```
  
<a href="http://docs.sequelizejs.com/en/latest/docs/getting-started/" target="_blank"> Sequelize </a> is a promise-based ORM for Node.js and io.js. It supports the dialects PostgreSQL, MySQL, MariaDB, SQLite and MSSQL and features solid transaction support, relations, read replication and more.

<a href="https://www.npmjs.com/package/pg" target="_blank"> pg <a> is PostgreSQL client for node.js. Pure JavaScript and optional native libpq bindings and <a href="https://www.npmjs.com/package/pg-hstore" target="_blank"> pg-hstore </a> is a module for serializing and deserializing JSON data into Postgres hstore key/value pair format.

Next step is to create a table called `Tasks` in our Postgres database which has 3 columns: `id`, `title` and `completed`, last field is a boolean telling us if the task is completed or not.

### Creating the model

#### Sequelize configuration

First step we need to take is to configure Sequelize inside our project so it can connect to our database. We need to add the following snipped of code to the `www` file located in the `bin` folder of the project:

``` js
var models = require("../models");

models.sequelize.sync().then(function () {
    var server = app.listen(app.get('port'), function() {
        debug('Express server listening on port ' + server.address().port);
    });
});
```

So now the `www` file should look something like this:

``` js
#!/usr/bin/env node
var debug = require('debug')('node-sequelize-postgresql');
var app = require('../app');
var models = require("../models");

app.set('port', process.env.PORT || 3000);

models.sequelize.sync().then(function () {
    var server = app.listen(app.get('port'), function() {
        debug('Express server listening on port ' + server.address().port);
    });
});
```

Now lets create `models` folder inside our project and add `index.js` file inside it. This file will contain all the configuration and connection code so that Sequelize can connect to our Postgres database.
  

  
