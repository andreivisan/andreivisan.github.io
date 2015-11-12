---
layout: post
title: Android Volley synchronous requests
published: false
---

### Introduction

<a href=""> Volley </a> is an HTTP library that makes networking for Android apps easier and most importantly, faster. One of the main features that Volley has is eliminating the need for using AsyncTask for network operations as well as allowing you to avoid using HttpUrlConnection and HttpClient. The main feature of Volley is that it is asynchronous which can lead to some issues when you want to do a synchronous request.

### The problem

One of the problems I faced using Volley was calling a webservice via its JsonObjectRequest inside an Android Library and wanting to return one of the values I received insinde the response. But wait ... you'll say ... can't you do it via Futures? 
Well, here comes the second problem. But first let's have a look how dealing with Futures while using Volley would look like:

``` java
int REQUEST_TIMEOUT = 10;

RequestFuture<JSONObject> future = RequestFuture.newFuture();
JsonObjectRequest request = new JsonObjectRequest(URL, null, future, future);
requestQueue.add(request);

//...

try {
  JSONObject response = future.get(REQUEST_TIMEOUT, TimeUnit.SECONDS); // this will block (forever)
} catch (InterruptedException e) {
  // exception handling
} catch (ExecutionException e) {
  // exception handling
}
```

Let's analyze the code above. What we do is to create a RequestFuture and we pass it as parameter to our JsonObjectRequest which then we add to the request queue.
Then the interesting part comes along. In order to get the response JSON object from our request we call the get() method on the future we declared above. We also set a timeout of let's say 10 seconds so we don't block the UI indefinitely in case our request times out.
But here's the catch - we can't call ``` future.get(REQUEST_TIMEOUT, TimeUnit.SECONDS) ``` on the main thread (the UI thread) as it is a blocking operation and it will result in a TimeoutException. So in order to solve this issue we need to call ``` future.get(REQUEST_TIMEOUT, TimeUnit.SECONDS) ``` in an AsyncTask, which is not something that we want.

### The code

One solution I came up with was to create a callback that I will pass to the method that has the call to JsonObjectRequest so that when we call this method we can use this callback to get the request value and process it further.

Let's start by creating a new Android project and add the following lines to the AndroidManifest.xml file:

``` xml
<uses-permission android:name="android.permission.INTERNET"/>

<application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:theme="@style/AppTheme"
        android:name=".controllers.NetworkController">
```


### Conclusion


