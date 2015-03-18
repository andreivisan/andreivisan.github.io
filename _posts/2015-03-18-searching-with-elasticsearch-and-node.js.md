---
layout: post
title: SEARCHING WITH ELASTICSEARCH AND NODE.JS
published: true
---

### Introduction
The following post will guide you through the creation of a project using Node.js, Express.js and search data that was previously indexed with Elasticsearch.
This tutorial assumes you already have installed Node.js and Express.js.
I will not describe how to index data. This is covered on Elasticsearch website and also on numerous other tutorials.

### Project creation
Let’s create a project using Express.js. Create a folder with the project name you desire.

``` bash
mkdir node_elasticsearch
```

Now let’s create the folder structure and files for our project. The parameter -e means that we will use EJS as template engine. Express defaults to Jade.

``` bash
cd node_elasticsearch
express -e
```

We now need to install all the modules from package.json with the following command:

``` bash
npm install
```

You can now run npm start and go to your browser of choice and type http://localhost:3000 and you should see the following:
Now let’s stop our app (Ctrl+C in the terminal) and let’s install Elasticsearch plugin for Node. 

### Elasticsearch plugin setup
We can do this by running the following command:

``` bash
npm install elasticsearch
```

Now we should create a new module for searching. If you use the MEAN stack (learn more about it <a href="http://mean.io/#!/">MEAN.IO</a> or <a href="http://meanjs.org/">MEAN.JS</a>) then there are many ways to organize your folder structure. Since we are not using this stack and we work with plain Node.js I prefer to organize each piece of different functionality in a new module. So let’s create a new folder called search_module in the root folder of the project. Inside it 
let’s create a file called search.js. Bellow the contents of the file:

``` javascript
var elasticsearch = require('elasticsearch');

var client = elasticsearch.Client({
  hosts: [
    ‘YOUR_ELASTICSEARCH_URL’
  ]
});

module.exports.search = function(searchData, callback) {
  client.search({
    index: ‘YOUR_INDEX_NAME’,
    type: 'url',
    body: {
      query: {
        bool: {
          must: {
            match: {
              "description": searchData.housingCityInput
            }
          },
          should: {
            match: {
              "url": searchData.housingCityInput
            }
          }
        }
      }
    }
  }).then(function (resp) {
    callback(resp.hits.hits);
  }, function (err) {
      callback(err.message)
      console.log(err.message);
  });
}
```

Above I have used bool search query. You can find more about different types of search queries <a href> HERE </a>.
The next step is to call this module from the router and display the search results.

### Put things together
Let’s add the following method inside routes/index.js

``` javascript
router.post('/search-results', function(req, res) {
  searchModule.search(req.body, function(data) {
    res.render('index', { results: data });
  });
});
```

Now let’s modify views/index.ejs to display our search results. The file should look like this:

``` html
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
    <h1><%= title %></h1>
    <form action='/search-results' method='post'>
      <input type="text" name="searchTerm" placeholder="your search term here">
      <button type="submit"> SEARCH </button>
    </form>
    <ul>
      <% if(locals.results) { %>
        <% results.forEach( function( result ) { %>
          <li>
            <%= result._source.title %>
          </li>
        <% }) %>
      <% } %>
    </ul>
  </body>
</html>
```

I indexed some houses for sale inside Elasticsearch and the screen looks like the following after I search for a city name:
You can index your own stuff and replace the result properties with the ones appropriate for you.
I hope this was helpful. Let me know bellow in the comments if there are any questions. 
You can find the code on GitHub <a href="https://github.com/andreivisan/node_elasticsearch">here</a>.
