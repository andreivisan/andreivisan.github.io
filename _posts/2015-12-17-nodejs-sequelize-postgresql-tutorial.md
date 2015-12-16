---
layout: post
title: Node.js, Sequelize, PostgreSQL tutorial
published: false
---

### Introduction

In this simple tutorial I will show you how to setup and start using Node.js together with Sequelize and PostgreSQL. These are very 
popular technologies, yet not so widely covered and properly explained how to use together. I will not cover the benefits of each of these technologies since they are very well documented independently. What I will do is show you how to use them together by creating the already infamous To Do List project.

### Configuration

To start I recommend using <a href="http://postgresapp.com/" target="_blank"> Postgress.app </a> if you are a Mac user, or just install PostgreSQL if you use another OS.
Next, lets create a Node.js project and install the following modules to work with Sequelize and PostgreSQL:

  ```
  npm install sequelize --save
  npm install pg --save
  npm install pg-hstore --save
  ```
  
<a href="http://docs.sequelizejs.com/en/latest/docs/getting-started/" target="_blank"> Sequelize </a> is a promise-based ORM for Node.js and io.js. It supports the dialects PostgreSQL, MySQL, MariaDB, SQLite and MSSQL and features solid transaction support, relations, read replication and more.

<a href="https://www.npmjs.com/package/pg" target="_blank"> pg <a> is PostgreSQL client for node.js. Pure JavaScript and optional native libpq bindings and <a href="https://www.npmjs.com/package/pg-hstore" target="_blank"> pg-hstore </a> is a module for serializing and deserializing JSON data into Postgres hstore key/value pair format.
  

  
