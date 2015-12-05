---
layout: post
title: The power of Observer pattern for Android UI libraries
published: false
---

### Introduction

### The Observer pattern

### The library

Our UI library will export an EditText field and a simple button that will post whatever we choose to write in our EditText. This is a simplified version of adding a post or a comment using a pre-defined component insinde a library to add the text we wish to post. The main purpose of this example is to emphasize the value of the Observer pattern when building a UI library.

First thing we need to create to get started is an Android project with an empty activity. Once this is done then we will create an Android library as a module inside our project. In Android Studio we can achieve this by right clicking on the project name and then New -> Module. In the selection window that pops up we will select Android library. I took the liberty to call mine just `uilibrary`.

Now if we take a look at our `settings.gradle` file inside our project we'll see that our library module was added as well. In order to be able to use it in our client project we will edit our `build.gradle` insinde the client project by adding the following line: `compile project(':uilibrary')`.

#### Custom View

#### Custom EditText

### The client
