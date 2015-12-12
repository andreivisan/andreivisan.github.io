---
layout: post
title: Node.js crawler using simplecrawler and cheerio
published: false
---

### Intorduction

In this post I will show you how to use two very popular Node.js modules in order to create a web crawler and also how to parse the data that you have crawled and structure it the way you want.
One of the main motivations for writing this post was the fact that I had to build a pretty complex web crawler and I found that <a href="https://github.com/cgiffard/node-simplecrawler" target="_blank"> simplecrawler </a> is an awesome Node.js module but there are not so many tutorials on how to use it besides the documentation provided by the author.
You can find the full code for this tutorial on <a href="https://github.com/andreivisan/node-crawler" target="_blank"> my Github </a> account.

### Installing necessary packages

Let's start by installing the necessary packages for our crawler. First one will be <a href="https://github.com/cgiffard/node-simplecrawler" target="_blank"> simplecrawler </a>. Simplecrawler is designed to provide the most basic possible API for crawling websites, while being as flexible and robust as possible. It has a lot of usefull events that can help you track the progress of your crawling process. In order to install this module you need to run `npm install --save simplecrawler` inside your project folder.
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

### The parser

### Conclusion
