---
layout: post
title: Improving Android Testing with Service Locator Design Pattern
published: false
---

### Introduction

Unit testing in Android is definitely a very challenging task. I find unit testing to be a critical component in raising the standard for the quality of any Android app. Besides being an Android developer, I am an Android user and I can objectively say that the quality of a big number of apps in the Google Play store is not as high as it should be and I think that a lot has to do with unit testing or the lack thereof.

The biggest challenge I found while writing unit tests in Android is injecting mock objects into Activities. There are a lot of cases where you build a service class to perform certain operations that you want to isolate from the Activity code and use in multiple places. This makes it very difficult for when, let's say, you want to test the functionality of the Activity and ignore the calls to the service methods. This can be achieved by injecting the mock service in the Activity, but in Android that is close to imposible since Android manages the creation and destruction of all activity instances, thereby making it difficult to inject dependencies. 

The solution that I came up with was to implement the Service Locator Design Pattern in order to set which service instance I want to use depending on the context.

Another solution would be to use <a href="http://square.github.io/dagger/" target="_blank"> Dagger </a> in order to make use of Dependency Injection, but I usualy prefer to add as few external libraries as possible in my apps.

### Service Locator Design Pattern

As per Wikipedia definition, the Service Locator Pattern is a design pattern used in software development to encapsulate the processes involved in obtaining a service with a strong abstraction layer. This pattern uses a central registry known as the "service locator", which on request returns the information necessary to perform a certain task.

### Implementation

Let's assume we have a service interface that I will call `Service.java` and an implementation of it called `ServiceImpl.java`.

``` java 
public interface Service {
    void fetchInformation(String url, final MyCallback callback);
}
```



