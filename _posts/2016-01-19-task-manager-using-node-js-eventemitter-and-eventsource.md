---
layout: post
title: Task manager using Node.js EventEmitter and EventSource
published: false
---

### Introduction

In this post we will create a task manager app, that will hold a list of tasks which we can start and stop. We will also have 2 attributes, `started at` and `finished at`. When we start a task a timer will be started and updated every second so you can see how much time you spent on one task and also check which task is on going. For this demo I will hold a list of tasks in a `.json` file since the purpose of the Task manager is to show you how you can elegantly interact with `EventSource` using Node's `EventEmitter`.

You can find the code for this project on <a href="https://github.com/andreivisan/node-event-source" target="_blank"> my GitHub account </a>.

In the end the application should look like the image bellow: 

![Merge sort](/public/images/TaskManager.png)

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


