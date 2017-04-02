---
layout: post
title: Task manager using Node.js EventEmitter and EventSource
published: true
---

### Introduction

In this post we will create a task manager app, that will hold a list of tasks which we can start and stop. We will also have 2 attributes, `started at` and `finished at`. When we start a task a timer will be started and updated every second so you can see how much time you spent on one task and also check which task is on going. For this demo I will hold a list of tasks in a `.json` file since the purpose of the Task manager is to show you how you can elegantly interact with `EventSource` using Node's `EventEmitter`.

You can find the code for this project on <a href="https://github.com/andreivisan/node-event-source" target="_blank"> my GitHub account </a>.

In the end the application should look like the image bellow: 

![Merge sort](/public/images/TaskManager.png)

### EventSource explained

The `EventSource` interface is used to receive server-sent events. It connects to a server over HTTP and receives events in text/event-stream format without closing the connection. So, in a way, it acts like a web socket by keeping an open communication with the server until this connection is specifically closed.

An important fact that is worth mentioning is that `EventSource` is not supported on IE (Internet Explorer). So, in case you want to use something similar that is compatible with IE as well, then web sockets are the safe bet.

### The code

This time we will start to look at the code from the UI to the server. So after starting a standard Express project let's start by editing `index.ejs` to look like the snippet bellow:

``` html
<!DOCTYPE html>
<html>
  <head>
    <title><%= title %></title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
    <h1><%= title %></h1>
    
    <div class="tasks-table-style">
        <table>
            <tr>
                <td>
                    Task
                </td>
                <td >
                    Started at
                </td>
                <td>
                    Finished at
                </td>
                <td>
                    Actions
                </td>
            </tr>
            <% if(tasks) { %>
                <% for(var i=0; i<tasks.length; i++) { %>
                    <tr>
                        <td>
                            <%= tasks[i].name %>
                        </td>
                        <td id="started-at-<%=i%>">
                            
                        </td>
                        <td id="finished-at-<%=i%>">
                            
                        </td>
                        <td>
                            <a href="#" onclick="startTimer(<%=i%>)"> [Start]</a>
                             | 
                            <a href="#" onclick="stopTimer()"> [End] </a>
                        </td>
                    </tr>
                <% } %>
            <% } %>
        </table>
    </div>
    
    <script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
    <script src="javascripts/ui.js"></script>
  </body>
</html>
```

So what we did was to create a table that will hold the tasks and added `jQuery` support and imported a `ui.js` file that will handle all the `js` code. Don't worry about the `css` class on the `div` that holds the table as you can find it all in the <a href="https://github.com/andreivisan/node-event-source" target="_blank"> GitHub repository </a>. Now let's have a look at the `ui.js` that holds the implementations for `startTimer(index)` and `stopTimer()` functions that are called on `[Start]` and `[Stop]` actions.

``` js
var startTaskEvent;

function startTimer(index) {
    $("#started-at-"+index).html((new Date()).toLocaleTimeString());
    startTaskEvent = new EventSource('/start-task');
    startTaskEvent.onmessage = function(e) {
        $("#finished-at-"+index).html(e.data);
    }
}

function stopTimer() {
    startTaskEvent.close();
}
```

As you can see in the code above, I created a new `EventSource` that points to an URL on the server. At this point the communication between the server and the client is opened. When the server will sent a message on the `response` object, the `onmessage` function is triggered and, in our case, we will add the data that we receive the `Finished at` column.

On the `stopTimer()` function the communication with the server is closed.

Now that the UI is done, let's move to the server side, where we will start by creating a custom `EventEmitter` that will send the response to the client. For the `EventEmitter` I created a new module called `task-events`.

``` js
var EventEmitter = require('events');
var util = require('util');

function TaskEmitter() {
    EventEmitter.call(this);
}

util.inherits(TaskEmitter, EventEmitter);

var taskEmitter = new TaskEmitter();

taskEmitter.on('startTimer', function(res) {
    res.writeHead(200, {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive'
    });
    
    setInterval(function() {
        res.write("data: " + (new Date()).toLocaleTimeString() + '\n\n');    
    }, 1000);
});

module.exports = taskEmitter;
```

As you can see in the code above I created a custom `EventEmitter` that responds on the `startTimer` event. When the `startTimer` event is triggered a response with the current time is sent every second to the client.

For the final bit, let's check the code in the `routes/index.js`.

``` js
var express = require('express');
var router = express.Router();
var fs = require("fs");

var taskEvents = require('../task-events')

/* GET home page. */
router.get('/', getTasks, function(req, res) {
  res.render('index', { title: 'Task manager' , tasks: req.tasks.tasks});
});

router.get('/start-task', function(req, res) {
    taskEvents.emit('startTimer', res);
});

function getTasks(req, res, next) {
    var filePath = "tasks.json";
    var fileContent = fs.readFileSync(filePath);
    req.tasks = JSON.parse(fileContent);
    next();
}

module.exports = router;
```

In the code above I created a middleware function that extracts all the tasks from the `tasks.json`. Then on `/start-task`, which is the URL that our `EventSource` called in order to open communication with the server, we emit the `startTimer` event on which we then send the response as shown above.

### Conclusion

By using `EventEmitter` to communicate with `EventSource` we created an elegant and easy to extend solution. Feel free to download the code from the <a href="https://github.com/andreivisan/node-event-source" target="_blank"> GitHub repository </a> and start the app to see it in action.

Please leave a comment bellow if you found this post useful or if I can offer further help.


