---
layout: post
title: Learning Spring Boot by building a concrete project - Part 1
published: false
---

### Introduction

In this tutorial we will go through Spring Boot and other Spring popular concepts by creating a practical project. The goal is that by the end of this tutorial we will have a full featured web application and you will be familiar with the most important Spring concepts.

### The web application

The web application that we will build in this tutorial will be one that manages your personal finances. Is an idea I have for a while, because in all honesty, managing personal finances has always been a challenge for me. 
I know the popular apps out there that already handle that for you, from the mobile ones to the web ones, but I prefer something simple that everyone can customize based on their personal preferences and needs.

#### Web application features

1. Add expenses
    1. Add expenses manually
    2. Add a CSV report of your expenses - most banks allow you to export a CSV file with your expenses. If your bank uses a different format feel free to fork the repository and add support for more file types.
2. Create expenses categories - The web application will allow to manually add new categories and rules, based on which expenses imported in bulk via CSV file upload will automatically be categorized.
3. Create budgets - The web application will allow to set budgets and limits for each budget.

If you think of any other cool features for this application feel free to add fork the repository and add them yourself, or suggest them in the comments bellow and we may add them in a future tutorial.

### Spring Boot

Spring Boot makes it easy to create stand-alone, production-grate Spring based applications that you can just run. Most Spring Boot applications need very little Spring configuration. Most popular features of Spring Boot are, the ability to create stand-alone Spring applications, embedded Tomcat, Jetty or Undertow without needing to deploy wars. 

Spring offers good documentation that you can follow <a href="https://projects.spring.io/spring-boot/“ target=“_blank”> here </a>.

### Getting started

Spring Boot 2.0.0.BUILD-SNAPSHOT requires Java 8 and Spring Framework 5.0.0.BUILD-SNAPSHOT or above. Explicit build support is provided for Maven (3.2+), and Gradle 3 (3.4 or later).

You can find many ways of installing Spring Boot href="https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started“ target=“_blank”> here </a> but for this tutorial we will use Spring Initializr, which is an online tool that generates your Spring Boot project and all you need to do is import it in you IDE.

Open href="https://start.spring.io“ target=“_blank”> Spring Initializr </a> page. Click on the ```Switch to full version``` link so that you can have an overview of the list of option from which you can pick in order to generate your project.

In the ```Group``` section fill in you desired package name (E.g. io.programminglife), select your artifact name (E.g. finance) and for dependecies I chose Web, Security, JPA, Actuator, Thymeleaf and Session. If we need more we can manually add more to our pom.xml file. Press on ```Generate project``` and you should have a folder downloaded in your preferred downloads location. All you have to do now is just import it in your IDE of choice and you have a fully running Spring Boot project.

In the next post we will start coding our actual logic for our project so stay tuned for that. In the meantime you can find the sources for this tutorial on href="https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started“ target=“_blank”> my GitHub </a>.
