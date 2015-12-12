---
layout: post
title: Node.js crawler using simplecrawler and cheerio
published: false
---

### Introduction

In this post I will show you how to use two very popular Node.js modules in order to create a web crawler and also how to parse the data that you have crawled and structure it the way you want.
One of the main motivations for writing this post was the fact that I had to build a pretty complex web crawler and I found that <a href="https://github.com/cgiffard/node-simplecrawler" target="_blank"> simplecrawler </a> is an awesome Node.js module but there are not so many tutorials on how to use it besides the documentation provided by the author.
You can find the full code for this tutorial on <a href="https://github.com/andreivisan/node-crawler" target="_blank"> my Github </a> account.

### Installing necessary packages

Let's start by installing the necessary packages for our crawler. First one will be <a href="https://github.com/cgiffard/node-simplecrawler" target="_blank"> simplecrawler </a>. Simplecrawler is designed to provide the most basic possible API for crawling websites, while being as flexible and robust as possible. It has a lot of useful events that can help you track the progress of your crawling process. In order to install this module you need to run `npm install --save simplecrawler` inside your project folder.
Second module we will install is <a href="https://github.com/cheeriojs/cheerio" target="_blank"> cheerio </a>. Cheerio is a fast, flexible, and lean implementation of core jQuery designed specifically for the server. We will use cheerio to parse the content of the crawled data and to structure it in any way we want. In order to install cheerio you need to run `npm install --save cheerio` inside your project folder.

### The crawler

For the crawler I created a module inside my project called `crawler` inside which I created a file called `index.js` containing the code bellow:

``` js
var Crawler = require('simplecrawler');

module.exports.crawl = function(callback) {
  var myCrawler = Crawler.crawl("http://programminglife.io/");

  var urls = [];

  myCrawler.on("crawlstart", function() {
    console.log("Crawler started!");
  });

  myCrawler.on("fetchcomplete", function(queueItem, responseBuffer, response) {
      callback(responseBuffer);
  });

  myCrawler.start();
}
```

The purpose of this crawler is to crawl my blog and get all the information on the first page. As you can see it is very easy to initialize, all you have to do is to call the crawl function with the `URL` as parameter. There are multiple ways to initialize it, but I found this way the best since it allows me to further configure it. You can also set crawler's depth by using `myCrawler.maxDepth` which by default is 0, or the interval by using `myCrawler.interval` or the number of processes you want to start for crawling by using `myCrawler.maxConcurrency`. There are many others parameters you can use which you can find on the <a href="https://github.com/cgiffard/node-simplecrawler" target="_blank">simplecrawler page</a>.

Simple crawler also has a lot of events you can call to check the status of the crawler. For this example I used `crawlstart` which marks the start of the crawling process and `fetchcomplete` which is fired when the resource is completely downloaded. There are many others you can use and you can find them on the <a href="https://github.com/cgiffard/node-simplecrawler" target="_blank">simplecrawler page</a>.

On `fetchcomplete` I triggered a callback function with the `responseBuffer` (which is the content of the crawled page) as parameter. I will use this to feed the scrapper in order to get the information I need from the crawled page.

### The scrapper

The scrappers's role will be to get the page that the crawler downloaded, parse it and extract the title and the date of each blog post from my blog.

For the scrapper I created another module called `scrapper` which contains 2 files. One is `index.js` which contains the configuration file called `scrapper-config.js`. The configuration file contains the elements description of the information I want to extract.

``` json
{
  "article" : {
    "base": ".archive",
    "title": ".col .span_8",
    "date": ".post-date time"
  }
}
```

So, as you can see above I set an element called `base` which holds the class of the base element containing all my articles. If you use `Firebug` or `Developer tools` you can check the homepage of my blog and see that the element that holds all my posts is a `selection` tag with class `.archive`. I applied the same logic for the title and the date.

Bellow is the code for the scrapper found in `scrapper/index.js`:

``` js
var cheerio = require('cheerio');

module.exports.extractData = function(html, config) {

  var $ = cheerio.load(html);

  $(config.article.base).each(function() {
    $(config.article.title, this).each(function(index, element) {
      console.log("Post title: %s", element.attribs.title);
    });

    $(config.article.date, this).each(function(index, element) {
      console.log("Post date: %s", element.attribs.datetime);
    });

  });
}
```

For the function above, the `html` parameter represents the information downloaded by the crawler and the `config` parameter represents the `JSON` object found int `scrapper-config.json`. 

### Putting it all together

Now that both the crawler and the scrapper are ready let's modify the `routes/index.js` file to look like the code bellow:

``` js
var express = require('express');
var router = express.Router();

var fs = require("fs");

var crawler = require('../crawler/index');
var scrapper = require('../scrapper/index')

/* GET home page. */
router.get('/', function(req, res) {
  var filePath = "scrapper/scrapper-config.json";
  var fileContent = fs.readFileSync(filePath);

  crawler.crawl(function(content) {
    scrapper.extractData(content, JSON.parse(fileContent));
  });
  res.render('index', { title: 'Express' });
});

module.exports = router;
```

Now, if you run the crawler by executing `npm start` in your project's folder you'll see a list of titles and dates representing the titles of the blog posts on my blog and the date each one was released.

### Conclusion

This is a very simple and basic web crawler that can be the base for any more complex crawler. Please comment bellow if you found this post useful, if you have any suggestions regarding this posts or if you need further help on this topic. You can find the full code for this tutorial on <a href="https://github.com/andreivisan/node-crawler" target="_blank"> my Github </a> account.
