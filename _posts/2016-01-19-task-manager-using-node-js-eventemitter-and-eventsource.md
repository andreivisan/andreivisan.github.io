---
layout: post
title: Task manager using Node.js EventEmitter and EventSource
published: false
---

### Introduction

In this post we will create a task manager app, that will hold a list of tasks which we can start and stop. We will also have 2 attributes, `started at` and `finished at`. When we start a task a timer will be started and updated every second so you can see how much time you spent on one task and also check which task is on going. For this demo I will hold a list of tasks in a `.json` file since the purpose of the Task manager is to show you how you can elegantly interact with `EventSource` using Node's `EventEmitter`.

You can find the code for this project on <a href="https://github.com/andreivisan/node-event-source" target="_blank"> my GitHub account </a>.


