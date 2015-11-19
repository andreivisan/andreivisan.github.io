---
layout: post
title: Android media server with Node.js web client
published: false
---

### Introduction

In this post I am going to show you how you can creade a media server on your Android device and stream media files, images and videos in our case, to a web client. The web client will be built using Node.js.

#### Motivation

One may wonder how would an application like this help or which use cases it may serve. There are a lot of cases where people like to build their own media servers and clients in order to store and manage their personal photos and videos. There are multiple ways in which this task may be achieved with the help of different cloud solutions out there, like Dropbox or Google Drive. But there are cases where you don't want your media in the cloud and you prefer to have it all stored on your personal server and just sending them via mail just won't do it for you. 

#### Technologies

In order to achieve this result I have used AndroidAsync for the server as well as Apache commons IO and Node.js with Express, ejs and request module.

### The Server

#### The Server - Expose media files

#### The Server - Stream images

#### The Server - Stream video

### The Client

#### The Client - Display files

#### The Client - Render images and videos

### Conclusion
