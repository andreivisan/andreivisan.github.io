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

Now lets create `models` folder inside our project and add `index.js` file inside it. This file will contain all the configuration and connection code so that Sequelize can connect to our Postgres database. Bellow is the complete code for `/models/index.js` : 

``` js
"use strict";

var fs        = require("fs");
var path      = require("path");
var Sequelize = require("sequelize");
var sequelize = new Sequelize(process.env.POSTGRESQL_LOCAL_DB, "", "", {
    host: process.env.POSTGRESQL_LOCAL_HOST,
    dialect: 'postgres',
    dialectOptions: {
        ssl: true
    },
    freezeTableName: true,
    pool: {
        max: 9,
        min: 0,
        idle: 10000
    }
});

var db = {};

fs
    .readdirSync(__dirname)
    .filter(function(file) {
        return (file.indexOf(".") !== 0) && (file !== "index.js");
    })
    .forEach(function(file) {
        var model = sequelize["import"](path.join(__dirname, file));
        db[model.name] = model;
    });

Object.keys(db).forEach(function(modelName) {
    if ("associate" in db[modelName]) {
        db[modelName].associate(db);
    }
});

db.sequelize = sequelize;
db.Sequelize = Sequelize;

module.exports = db;
```

The code above contains the configuration to our Postgres database as well as it loads the models that we'll implement in our `models` module. Important to notice in the code above are the following settings:

`dialectOptions: { ssl: true }` - This setting activates SSL for Postgres connections. It is very useful when you connect to an instance on Heroku or Amazon WS. If you connect to `localhost` then this has to be false.

`freezeTableName: true` - This setting freezes the table name to what you define in your model file as you will see later in the code. Normaly Sequelize tends to add an "s" to the end of the table name that you define in your code if the table name you defined doesn't contain one already. If your table in your Postgres database does contain an "s" at the end then it's all fine, you don't need this setting, but I would use it anyway just for safety measures.

#### Creating the model

Now it's time to create the model. Inside `models` module lets create a file called `tasks.js` that looks like the code snippet bellow:

``` js
"use strict"

module.exports = function(sequelize, DataTypes) {
    var Tasks = sequelize.define("Tasks", {
        title: {
            type: DataTypes.STRING,
            allowNull: false
        },
        completed: {
            type: DataTypes.BOOLEAN,
            allowNull: false
        }
    }, {
       tableName: 'Tasks'
    });

    return Tasks;
}
```

  
