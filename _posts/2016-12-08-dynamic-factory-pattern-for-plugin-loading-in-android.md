---
layout: post
title: Dynamic Factory Pattern for Android
published: false
---


### Introduction

The Dynamic Factory Pattern describes a factory that can create service instances based on concrete type definitions stored in external plugins which can be later loaded without having to modify the factory class or the core implementation.

### Context

You are building a framework that is composed of different modules that can be fully or partially included in a project. Your framework core is not aware of the contents of the underlying modules in order to offer full flexibility and modularity. The implementing system should be able to implement the framework and later add new modules without having to change the core implementation of the framework.

### Example


